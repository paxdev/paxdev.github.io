---
  title: Powershell Functions
  categories:
    - powershell
  tags:
    - powershell
---

* Create a function: 
  ```powershell
  Function SomeFunction {
      # Script ...
  }
  ```
* Pass parameters
  ```powershell
  param(
      [type1]$param1, 
      [type2]$param2
  )
  ```
  * Use as the first line of a function (after the opening curly brace) to pass params to a function
  * Use as the first line of a script to capture command line arguments
  * Normal parameters can be specified `-paramName value`
  * Use `[switch]$someFlag=$false` to allow a "flag" type switch `-someFlag`
