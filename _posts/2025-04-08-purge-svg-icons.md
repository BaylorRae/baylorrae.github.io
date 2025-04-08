---
layout: post
title: Purge SVG icons in Rails deployment
date: 2025-04-08 08:00:00 -0500
categories:
- Rails
tags:
- rails
---

Over the last few months I've enjoyed using [Heroicons](https://heroicons.com/) and [Phosphor](https://phosphoricons.com/) icon sets.  These icons have variants at multiple sizes, are squares, and I don't require loading a font library.  I guess things might change as FontAwesome 7 rolls out, but that hasn't happened yet.

### The Setup

My setup has been in contant flux but lately I've started copying all the icon files into `app/assets/images` and rendering them with [`inline_svg`](https://rubygems.org/gems/inline_svg) and a custom helper.

```ruby
module ApplicationHelper
  def svg(name, **args)
    return if name.blank?
    filename = "#{name}.svg" unless name.end_with?(".svg")
    inline_svg(filename, **args)
  end
end
```

The icons are stored with in directories based on the icon source and variant.

```bash
$ find app/assets/images -name "*.svg"
# app/assets/images/fontawesome/regular/*.svg
# app/assets/images/heroicons/micro/*.svg
# app/assets/images/heroicons/mini/*.svg
# app/assets/images/heroicons/outline/*.svg
# app/assets/images/heroicons/solid/*.svg
```

Using this helper and this file structure makes it quick to add icons to views and in other helper methods.

```erb
<%= link_to post, class: 'flex items-center' do %>
  <span>View More</span>
  <%= svg "heroicons/micro/chevron-right", class: 'size-4 ml-1' %>
<% end %>
```

### The Problem

One of the projects I'm working on right now has almost 1,600 svg icons saved into the project.  I'm perfectly happy with this during development but it is very slow during deployment when precompiling assets.

The solution I came up with the help of ChatGPT :) was to remove the icons that are not referenced in `app/controllers`, `app/helpers`, and `app/views`.  This script purges 1,500+ icons in less than a second and allows me to be fast during development and pushing to production.

```bash
#!/usr/bin/env bash

set -euo pipefail

namespaces=("heroicons" "fontawesome" "phosphor")
pattern="$(IFS='|'; echo "${namespaces[*]}")"

# Step 1: Find all SVG files under the selected namespaces
mapfile -t svg_files < <(find app/assets/images -type f -name "*.svg" | grep -E "$pattern")

# Step 2: Extract all possible used keys from views using grep
# This builds a list like: heroicons/outline/arrow-left, etc.
mapfile -t used_keys < <(grep -rhoE "($pattern)/[a-zA-Z0-9_-]+/[a-zA-Z0-9_-]+" app/controllers app/helpers app/views | sort -u)

# Step 3: Put keys into a lookup set (associative array for fast matching)
declare -A used_lookup
for key in "${used_keys[@]}"; do
  used_lookup["$key"]=1
done

# Step 4: Check all SVGs in one loop (still O(n), but lookup is O(1))
unused_files=()
for path in "${svg_files[@]}"; do
  key="${path#app/assets/images/}"
  key="${key%.svg}"

  if ! [[ ${used_lookup["$key"]+_} ]]; then
    unused_files+=("$path")
  fi
done

# Step 5: Delete unused files in batches (fast)
if [[ ${#unused_files[@]} -gt 0 ]]; then
  printf "%s\n" "${unused_files[@]}" | xargs -P 4 rm
fi
```