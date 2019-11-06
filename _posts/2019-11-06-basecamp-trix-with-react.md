---
layout: post
title: Basecamp's Trix with React
date: 2019-11-06 08:00:00 -0500
categories:
- React Hooks
- Trix Editor
tags:
- react-hooks
- trix-editor
---

I needed to add the Trix Editor to my React application and couldn't find a
simple solution. Since having an actual `<input />` node in the DOM isn't
required, I opted to have the value bound directly the editor's built in input
node.

### Usage

```js
import React, { useState } from 'react'

const Form = () => {
  const [body, setBody] = useState('')

  return (
    <Editor value={body} onChange={setBody} />
  )
}
```

### The `Editor` Component

```js
import React, { useEffect, useRef } from 'react'
import 'trix'
import 'trix/dist/trix.css'

const Editor = ({ value, onChange }) => {
  const trixEditor = useRef(null)

  useEffect(() => {
    trixEditor.current.addEventListener('trix-change', (e) => {
      onChange(trixEditor.current.value)
    })
  }, [trixEditor])

  useEffect(() => {
    if (!trixEditor.current) return
    if (trixEditor.current.inputElement.value === value) return
    trixEditor.current.value = value
  }, [value])

  return (
    React.createElement('trix-editor', {ref: trixEditor})
  )
}

export default Editor
```
