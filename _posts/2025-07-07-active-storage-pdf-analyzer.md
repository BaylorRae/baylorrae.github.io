---
layout: post
title: ActiveStorage PDF Analyzer
date: 2025-07-07 08:00:00 -0500
categories:
- Rails
tags:
- rails
- active_storage
---

I found this code while looking through some EDA project folders on my laptop.
It adds a PDF analyzer to ActiveStorage by checking the mime type of the
uploaded file for `application/pdf` and then attaches the metadata from the
`pdf-reader` gem.

## 1. Add the `pdf-reader` gem to your Gemfile

```ruby
gem "pdf-reader", "~> 2.14"
```

## 2. Add the analyzer

```ruby
# /lib/pdf_analyzer.rb

class PdfAnalyzer < ActiveStorage::Analyzer
  def self.accept?(blob)
    blob.content_type == "application/pdf"
  end

  def metadata
    download_blob_to_tempfile do |file|
      reader = PDF::Reader.new(file)

      reader.info
        .transform_keys { |key| key.to_s.underscore }
        .transform_values { |value| parse_pdf_datetime(value) }
        .merge({
          page_count: reader.page_count
        })
    end
  end

  private

  def parse_pdf_datetime(time)
    match = time.match(/D:(\d{14})([+-]\d{2})'(\d{2})'/)

    if match
      datetime_str, offset_hours, offset_minutes = match.captures

      # Parse the datetime portion
      time = Time.strptime(datetime_str, "%Y%m%d%H%M%S")

      # Convert offset to ActiveSupport::TimeZone format
      offset_seconds = offset_hours.to_i * 3600 + offset_minutes.to_i * 60
      time_with_zone = time.in_time_zone(offset_seconds)

      time_with_zone
    else
      time
    end
  end
end
```

## 3. Register the analyzer with ActiveStorage

```ruby
# /config/application.rb

config.active_storage.analyzers << ::PdfAnalyzer
```

## Usage

```ruby
document = Document.first
document.file.metadata
# {
#     "identified"=>true,
#     "analyzed"=>true,
#     "producer"=>"PFU PDF Library 1.2.1",
#     "creator"=>"ScandAll PRO V2.1.5",
#     "creation_date"=>"2023-11-01T11:32:25.000-06:00",
#     "mod_date"=>"2023-11-01T11:32:25.000-06:00",
#     "metadata_date"=>"2023-11-01T11:32:25.000-06:00",
#     "page_count"=>16
#  }
```