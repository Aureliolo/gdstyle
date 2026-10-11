## gdstyle 0.3.3

A formatter fix release for files that use `@warning_ignore_start` and
`@warning_ignore_restore`. `fmt` no longer changes which code a region covers,
and it keeps the two markers tight against the region.

### Fixed

- **`fmt` no longer moves members into or out of a `@warning_ignore_start`
  region.** The two markers are standalone lines, and the reorder moved each
  one with whichever member followed it, so the pair did not move together:

  ```gdscript
  var health = 10
  @warning_ignore_start("unused_signal")
  signal a
  signal b
  @warning_ignore_restore("unused_signal")
  var z = 1
  ```

  0.3.2 sorted the signals above `health` and left the restore marker with
  `z`, so `health` ended up inside the region with its warnings silenced:

  ```gdscript
  @warning_ignore_start("unused_signal")

  signal a
  signal b

  var health = 10


  @warning_ignore_restore("unused_signal")
  var z = 1
  ```

  `fmt` now skips the reorder whenever it would change which lines a region
  covers, in the script and in inner classes. It still reorders when only
  members outside the region move. On a skipped file `check` keeps reporting
  `order/class-member-order`, so the order can be fixed by hand. The README
  has a new section, "When the formatter leaves the order alone", that lists
  the cases. ([#41](https://github.com/atelico/gdstyle/pull/41))

- **Region markers stay tight against the code they bracket.** The gap between
  members could land between `@warning_ignore_start(...)` and the first member
  it covers. The start marker now counts as the head of the first covered
  member (above its doc and comment block, if it has one), the restore marker
  as the tail of the last one, and blank lines between a marker and its region
  are removed. The example above now formats to:

  ```gdscript
  var health = 10

  @warning_ignore_start("unused_signal")
  signal a
  signal b
  @warning_ignore_restore("unused_signal")

  var z = 1
  ```

  Around functions, the two blank lines go above the start marker and below
  the restore marker. This applies to script-level markers; inner-class bodies
  keep their blank lines as written. **Files with region markers may be
  reformatted once.** ([#41](https://github.com/atelico/gdstyle/pull/41))

### Changelog

- Fixed: `fmt` no longer changes which members a `@warning_ignore_start` /
  `@warning_ignore_restore` region covers when it reorders (#41).
- Fixed: `fmt` keeps region markers tight against the region, with blank lines
  outside it (#41).
- Docs: new README section on when `fmt` skips reordering, and a
  `CHANGELOG.md` with a short entry for every release.

### Upgrading

No configuration changes are required. This resolves the known issue listed
in the 0.3.2 notes.
