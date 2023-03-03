---
layout: default
title: input form
parent: Flutter Widget Index
nav_order: 9
---

<br>

- [Form](https://api.flutter.dev/flutter/widgets/Form-class.html)
  An optional container for grouping together multiple form field widgets (e.g. TextField widgets).

  Each individual form field should be wrapped in a FormField widget, with the Form widget as a common ancestor of all of those.
  Call methods on FormState to save, reset, or validate each FormField that is a descendant of this Form. To obtain the FormState,
  you may use Form.of with a context whose ancestor is the Form, or pass a GlobalKey to the Form constructor and call GlobalKey.currentState.
  
  - keyboardType  
  - decoration
    - hintText
    - labelText
    - prefixIcon
    - suffixIcon
  - validator
  - onChanged
  - onSaved
  - onFieldSubmitted

- [FormState](https://api.flutter.dev/flutter/widgets/FormState-class.html)  
  State associated with a Form widget. A FormState object can be used to save, reset, and validate every FormField that is a descendant of the associated Form.

<br>

