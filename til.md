---
layout: page
title: TIL
page_main_title: Today I learned
permalink: /til/
---

## Immediate object types

*Oct 06, 2023 • Ruby*

Immediate objects(values) in Ruby are objects that do not require memory 
allocation to create and memory indirection to access, and as such they are
generally faster than non-immediate objects.

e.g. `true`, `false`, `nil`, `Fixnum`, `Symbol`

These objects are stored directly in `VALUE`.
