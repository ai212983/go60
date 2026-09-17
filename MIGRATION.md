# Glove80 -> Go60 port

Authoritative source of truth: <https://github.com/ai212983/glove80>,
`origin/main` commit `8c2660d31894c59b61eb14e97ef6211ae06041ac`
(read via `git show 8c2660d:path` in
`/Users/dimitri/Documents/Code/personal/glove80/glove80-firmware`;
neither source repo was modified).

Ported files: `config/glove80.keymap` -> `config/go60.keymap`,
`config/helper.h` and `config/international_chars/russian.dtsi` copied
byte-exact (verified against the commit by
`/tmp/verify-go60-authoritative.py`). `config/go60.conf`,
`config/default.nix`, and build tooling are untouched.

## Geometry and layers

Five layers, 60 keys each, in order Base0 Russian1 NavNum2 Magic3 Symbols4.
`layer_Factory` and `#define LAYER_Factory 4` were removed; the former
Magic-to-Factory switch key (`&to LAYER_Factory`, Magic main-row-4 right
outer) is now `&none`, and Magic is otherwise byte-identical to the prior
commit. Symbols reindexed from 5 to 4 (references stay named
`&lt LAYER_Symbols SPACE`). Go60 main rows map source rows 2-5
(dedicated F-row omitted);
lower extras map source bottom-row C4/C3/C2 (left) and C2/C3/C4 (right);
six thumbs map source T4/T5/T6 (left) and T6/T5/T4 (right).

## Relocations vs source (all other keys match source exactly)

- Base top-left: source `&none` -> `&magic LAYER_Magic 0`; Russian same
  seat is `&trans` (Magic reachable by fallthrough).
- Base/​Russian top-right number-row: source `&kp PSWD` (`#define PSWD F18`).
  Unused `HUE_TMP`/`HUE_BRG`/`HUE_PWR` definitions removed.
- Base lower extras: source `L_C4R5 &none` -> `&kp LCTRL`;
  source `R_C4R5 &none` -> `&tog 1` (`&tog LAYER_Russian`);
  remaining four are source LEFT/RIGHT/UP/DOWN. Russian lower extras fall
  through to Base, with `&to LAYER_Base` (`&to 0`) at the toggle seat.
- Base thumbs unchanged except `&lt 2 BSPC` -> `&lt LAYER_NavNum BSPC`
  and `&lt 3 SPACE` -> `&lt LAYER_Symbols SPACE` (same layers, named).
- Symbols number row: source F1..F10 kept, F11/F12 on spare outer keys.
- Symbols right-hand rows restored exactly from remote, inner->outer:
  PRV_SPC,NXT_SPC,LBKT,RBKT,LS(N6),APP_LNC;
  PRV_TAB,NXT_TAB,LS(N9),LS(N0),LS(N8),APP_SWT;
  AMPS,GRAVE,LS(LBKT),LS(RBKT),spare,MSN_CTL.
- Spare-key relocations (documented): plain `&kp LALT` (source extra thumb
  with no Go60 seat) -> Symbols outer left upper-letter-row key (source
  `&none`); plain `&kp LSHFT` -> Symbols spare right bottom-letter-row key
  (source `&none`, retained).

## Preserved from source

Alpha and Russian grids, `HOST_OS 2` Unicode setup with `helper.h` /
`russian.dtsi` includes, HYPR/MEH definitions, all host definitions and
mod-morphs (`dqt_sqt`, `gra_til`). Source Magic/macros/LED overrides not
imported. Obsolete template `keypad_td`/`symbol_nav_td` remain removed.

## Unchanged from template (Go60 template Magic retained, Factory removed)

`layer_Magic` (except the single removed Factory-switch key now `&none`),
supporting `magic`/`bt_*` behaviors, `macros`, `input_processors`, and the
right trackpad (`cirque_rh_listener`) config are byte-exact vs prior
`HEAD:config/go60.keymap`. `layer_Factory` and its `#define` were deleted.
The template left-trackpad `layer_4` (`layers = <4>`) Factory override was
removed so it cannot unintentionally activate on Symbols4; normal left
scroll/right-click behavior and right pointer 3/1 with the NavNum2
override 9/1 are retained.

## Local source working edits (not migrated)

The source working tree differs from the authoritative commit only by
Studio support: one `&studio_unlock` key (remote has `&none` there) plus
Studio build wiring. There are no Symbols differences vs remote.
Go60 retains its existing build settings; local Studio additions were not ported.

## Build record

No-Factory build succeeded in running container `go60-config-validation`
(source `8c2660d31894c59b61eb14e97ef6211ae06041ac`, firmware `/src` at
upstream `4b2f3b9b718a4508e87111cdec24cda16f3c2726`) with
`cd /config; nix-build ./config --arg firmware "import /src/default.nix
{}" -j2 -o /tmp/combined --show-trace` (host log
`/tmp/go60-no-factory-firmware-build.log`, exit 0).
`/tmp/combined/go60.uf2` copied to
`/Users/dimitri/Documents/Code/personal/go60/go60.uf2`
(sha256 `0ef7f08efa4935364c76c5b6a634dbb34e3b9af36cf85ae75dc43169cf0a8645`,
892416 bytes, UF2 magic and 512-byte blocks confirmed). Hardware untested.
