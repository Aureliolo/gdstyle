## gdstyle 0.3.2

A small release with one new formatter option and one formatter fix. `fmt` can
now keep a blank line between public and private variables, and it no longer
pushes `@warning_ignore(...)` away from the declaration it covers.

### Added

- **`blank_line_between_public_and_private` config setting** (default:
  `false`). `fmt` puts same-kind declarations in one tight block, so public and
  `_`-prefixed variables ran together and a blank line added by hand was
  removed on the next format. With the setting on, `fmt` keeps one blank line
  wherever a public declaration meets a private one:

  ```toml
  blank_line_between_public_and_private = true
  ```

  ```gdscript
  var is_idle: bool = false
  var idle_timer: float = 0.0

  var _velocity: Vector2 = Vector2.ZERO
  ```

  It applies to constants, static variables, `@export` variables, plain
  variables and `@onready` variables. Signals, enums and functions are not
  affected.

  Two limits to know about:

  - Members are not reordered. The line is added at every public/private
    boundary, so interleaved declarations (`a`, `_b`, `c`) get a separator at
    each one.
  - It applies to script-level members. Inner-class bodies keep the blank
    lines they were written with.

  Godot's style guide has no written rule for this, which is why it is off by
  default. Output is unchanged unless you turn it on.
  ([#38](https://github.com/atelico/gdstyle/issues/38),
  [#39](https://github.com/atelico/gdstyle/pull/39))

### Fixed

- **`fmt` no longer separates `@warning_ignore(...)` from its declaration.**
  An annotation on its own line was treated as the end of the previous member,
  so the blank lines between members were inserted below it. A doc comment
  above it was also split off from the declaration:

  ```gdscript
  var health = 10

  ## Cached speed.
  @warning_ignore("unused_private_class_variable")

  var _speed = 300.0
  @warning_ignore("unused_parameter")


  func foo(a):
  	pass
  ```

  The annotation now stays tight against the declaration below it, and the
  blank lines go above it:

  ```gdscript
  var health = 10
  ## Cached speed.
  @warning_ignore("unused_private_class_variable")
  var _speed = 300.0


  @warning_ignore("unused_parameter")
  func foo(a):
  	pass
  ```

  This covers variables, constants, signals, enums, functions and inner
  classes, including several stacked `@warning_ignore` lines. The old output
  still loaded in Godot, so nothing was broken at runtime. **Files with a
  standalone `@warning_ignore` line will be reformatted once.**
  ([#40](https://github.com/atelico/gdstyle/pull/40))

### Known issues

- When `fmt` reorders members that sit between `@warning_ignore_start` and
  `@warning_ignore_restore`, the two markers do not move as a pair, so the
  region can end up covering different members. This is not new in 0.3.2.

### Upgrading

No configuration changes are required. The new setting is opt-in. The only
change to existing output is the `@warning_ignore` fix above.
