---
layout: post
title: Process HEIC Images with Node on AWS Lambda
date: 2020-01-13 08:00:00 -0500
categories:
- Node
- AWS Lambda
- Sharp
- HEIC
- libvips
- libheic
tags:
- node
- aws-lambda
- sharp
- heic
- libvips
- libheic
---

I needed to add support for HEIF/HEIC images in my application and thought it
would be a fun introduction to AWS Lambda. Since I have minimal experience with
AWS and this was an exercise for myself I thought it would also be fun to use
[Sharp](https://sharp.pixelplumbing.com/en/stable/).

The first thing I learned is that HEIF is not shipped with image processing
libraries, ImageMagick or Sharp. From what I can tell this is mostly because of
licensing issues that thankfully don't apply to me. As a result, the first
requirement is to compile a [Vips](https://github.com/libvips/libvips) with a
self install version of [libheif](https://github.com/strukturag/libheif) and
[libde265](https://github.com/strukturag/libde265).

This is the largest hurdle since AWS Lambda runs application code in a
stateless, isolated environment. To overcome this we need to build and package
these libraries and attach them to our Lambda function. I use the word "attach"
since I uploaded these binaries as a Lambda layer to lighten my application
code.

I used a [multi-stage Dockerfile](#dockerfile) to build the binaries, copy them
into another isolated environment and install the sharp node module. I then
wrapped these steps in [a shell script](#compile-and-copy-script) to further the
portability of it.

### Dockerfile

```docker
# Use environment that closely matches AWS Lambda
FROM lambci/lambda:build-nodejs12.x AS builder

WORKDIR /var/task

# Download dependencies for vips, libheif and libde265
RUN curl -L https://github.com/libvips/libvips/releases/download/v8.9.0/vips-8.9.0.tar.gz | tar xz && \
    curl -L https://github.com/strukturag/libde265/releases/download/v1.0.4/libde265-1.0.4.tar.gz | tar xz && \
    curl -L https://github.com/strukturag/libheif/releases/download/v1.6.1/libheif-1.6.1.tar.gz | tar xz && \
    yum install -y gtk-doc gobject-introspection gobject-introspection-devel expat-devel libjpeg-turbo libjpeg-turbo-devel libpng libpng-devel

# Install h.265 video codec library
RUN cd libde265-1.0.4 && \
    ./autogen.sh && \
    ./configure && \
    make && \
    make install

# Install HEIF library
RUN cd libheif-1.6.1 && \
    ./autogen.sh && \
    PKG_CONFIG_PATH=/usr/local/lib/pkgconfig ./configure && \
    make && \
    make install

# Install vips
RUN cd vips-8.9.0 && \
    ./autogen.sh && \
    PKG_CONFIG_PATH=/usr/local/lib/pkgconfig ./configure --prefix=/var/task/vendor && \
    make && \
    make install

# Copy pkg-config files to working directory
RUN cp /usr/local/lib/pkgconfig/* vendor/lib/pkgconfig/ && \
    cp /usr/lib64/pkgconfig/libpcre* vendor/lib/pkgconfig/ && \
    cp /usr/lib64/pkgconfig/glib* vendor/lib/pkgconfig/ && \
    cp /usr/lib64/pkgconfig/gobject* vendor/lib/pkgconfig/ && \
    cp /usr/lib64/pkgconfig/gmodule* vendor/lib/pkgconfig/ && \
    cp /usr/lib64/pkgconfig/gthread* vendor/lib/pkgconfig/ && \
    cp /usr/lib64/pkgconfig/libpng* vendor/lib/pkgconfig/ && \
    cp /usr/lib64/pkgconfig/libjpeg* vendor/lib/pkgconfig/

# Copy compiled binaries to working directory
RUN cp /usr/local/lib/libheif.* vendor/lib/ && \
    cp /usr/local/lib/libde265.* vendor/lib/ && \
    cp /usr/lib64/libpng* vendor/lib/ && \
    cp /usr/lib64/libjpeg* vendor/lib/ && \
    cp -r /usr/lib64/glib*/include/* vendor/include/ && \
    cp -r /usr/include/glib*/* vendor/include/ && \
    cp -r /usr/lib64/gobject* vendor/lib/ && \
    cp /usr/lib64/libexpat* vendor/lib/ && \
    cp /usr/lib64/libgthread* vendor/lib/ && \
    cp /usr/lib64/libgmodule* vendor/lib/ && \
    cp /usr/lib64/libpcre* vendor/lib/ && \
    cp /usr/lib64/libgobject* vendor/lib/ && \
    cp /usr/lib64/libglib* vendor/lib/


# Starting in a new image ensures we have everything for
# an isolated environment.
FROM lambci/lambda:build-nodejs12.x

WORKDIR /var/task

# Copy results from builder image
COPY --from=builder /var/task/vendor ./vendor

# Install sharp with node
RUN LD_LIBRARY_PATH=/var/task/vendor/include PKG_CONFIG_PATH=/var/task/vendor/lib/pkgconfig npm install sharp
```

### Compile and Copy Script

The following script builds and runs the docker image and copies the necessary
files to the host machine. It's important to note that the files are copied to a
directory structure that is matches `$PATH`, `$NODE_PATH` and `$LD_LIBRARY_PATH`
inside the Lambda environment.

```bash
#!/usr/bin/env bash

main() {
  echo "Building Image"
  IMAGE_ID=$(docker build . -q)

  echo "Starting Container"
  CONTAINER_ID=$(docker run -d -it $IMAGE_ID bash)

  echo "Copying node_modules into node_modules"
  docker cp $CONTAINER_ID:/var/task/node_modules ./vips-sharp-layer-env/nodejs

  echo "Copying vendor into vendor"
  docker cp $CONTAINER_ID:/var/task/vendor/bin ./vips-sharp-layer-env
  docker cp $CONTAINER_ID:/var/task/vendor/lib ./vips-sharp-layer-env

  echo "Removing container"
  docker kill $CONTAINER_ID
  docker rm $CONTAINER_ID
}

main
```

### Creating the Lambda Layer

I'm using CloudFormation and the CDK to build out my application. The following
will upload the results from the above script as a Layer and add it to a Lambda
Function.

```ts
const uploadBucket = new s3.Bucket(this, "Upload")
const outputBucket = new s3.Bucket(
  this,
  "Output",
  {
    blockPublicAccess: BlockPublicAccess.BLOCK_ACLS,
  }
)

outputBucket.addToResourcePolicy(new iam.PolicyStatement({
  effect: iam.Effect.ALLOW,
  principals: [new iam.AnyPrincipal()],
  actions: ['s3:GetObject'],
  resources: [`arn:aws:s3:::${outputBucket.bucketName}/*`]
}))

const vipsSharpLayer = new lambda.LayerVersion(
  this,
  "VipsSharpLayer",
  {
    compatibleRuntimes: [lambda.Runtime.NODEJS_12_X],
    code: lambda.Code.fromAsset("lambda/process-images/vips-sharp-layer-env")
  }
)

const resizeHandler = new lambda.Function(
  this,
  "ResizeHandler",
  {
    runtime: lambda.Runtime.NODEJS_12_X,
    code: lambda.Code.fromAsset("lambda/process-images/process-images"),
    handler: "resize.handler",
    timeout: Duration.seconds(30),
    layers: [vipsSharpLayer],
    environment: {
      OUTPUT_BUCKET: outputBucket.bucketName
    }
  }
)

uploadBucket.grantRead(resizeHandler)
outputBucket.grantWrite(resizeHandler)

resizeHandler.addEventSource(new S3EventSource(uploadBucket, {
  events: [s3.EventType.OBJECT_CREATED],
}))
```

### The Lambda Function

This code barely scratches the surface of what I want it to do long term. Right
now it simply loads the S3 object into sharp, formats it to JPG and uploads it
into an output bucket.

I've also introduced [ExifReader](https://github.com/mattiasw/ExifReader) to
read the orientation of the image to ensure it is rotated correctly during the
upload. Sharp has support for auto rotating the image but I found it couldn't
read the orientation data from HEIC images. This is unfortunate as it requires
me to load the entire image into memory rather than stream it into the output
bucket.

```js
const AWS = require('aws-sdk')
const sharp = require('sharp')
const ExifReader = require('exifreader')

const S3 = new AWS.S3({
  maxRetries: 0,
  region: 'us-east-1'
})

const ORIENTATION_MAP = {
  auto: undefined,
  3: 180,
  6: 90,
  8: -90
}

exports.handler = async (event, context) => {
  const srcBucket = event.Records[0].s3.bucket.name;
  const srcKey = event.Records[0].s3.object.key;

  console.log(`Getting ${srcKey} from s3://${srcBucket}`)
  const imageObject = await S3.getObject({
    Bucket: srcBucket,
    Key: srcKey
  }).promise()

  const image = imageObject.Body

  console.log('Read exif data')
  let orientation = 'auto'
  try {
    const exif = ExifReader.load(image)
    orientation = exif && exif.Orientation && exif.Orientation.value || 'auto'
  }catch (e) {
    console.log('Failed reading exif data')
    console.log(e)
  }

  console.log('Building Sharp Pipeline \'toJPG\'')
  const output = sharp(image)
    .rotate(ORIENTATION_MAP[orientation])
    .toFormat('jpg')

  console.log('Upload image')
  const uploadData = await S3.upload({
    Body: await output.toBuffer(),
    Bucket: process.env.OUTPUT_BUCKET,
    ContentType: 'image/jpg',
    Key: srcKey.split('.')[0] + '.jpg'
  }).promise()

  console.log('Data: ', {
    ...uploadData
  })

  return {
    message: 'Success',
    timeRemainingMs: context.getRemainingTimeInMillis()
  }
}
```

And that's it. Aside from locating all the necessary binaries to create a
portable version of Vips it is pretty straight forward. Hopefully you can use
this as a reference for your own project and you findit helpful.
