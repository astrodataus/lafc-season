# LAFC · Black & Gold

Nine seasons of Los Angeles Football Club as a single interactive document.
Points against expected points, attack and finishing, squad contribution,
opponent records, rivalries, and attendance.

**Live site:** https://astrodataus.github.io/lafc-season/

Built by [Astrodata](https://astrodata.us). The same document also runs as an
Omni app; this repository is the static build of it.

Not affiliated with Los Angeles Football Club or Major League Soccer. Club marks
are placeholders.

---

## One file, no requests

`index.html` is the whole application: markup, styles, script, three typefaces,
the masthead photograph, and all 2,451 rows of data. Loading it makes **exactly
one request**, for the document itself.

There was never a chart library here. Every chart is hand-drawn SVG, which is
the house pattern and the reason this app survived an Omni outage that took the
others down. Two things did leave the origin and both are now inlined:

| Was | Now |
|---|---|
| Jost, Newsreader and JetBrains Mono from Google Fonts | 91 KB of subset woff2, base64 inlined |
| the masthead photograph from `upload.wikimedia.org` | 129 KB inlined, greyscale at 1400px |

The photograph sits at 34% opacity behind a scrim and is greyscaled and
sepia-toned in CSS, so storing it greyscale at 1400px rather than 1920px colour
takes it from 500 KB to 129 KB with nothing visible changing.

Total: 664 KB, **320 KB gzipped**.

### Credit

Masthead: the 3252 supporters' section, BMO Stadium, photograph by
BagmanTheEditor via Wikimedia Commons, **CC BY-SA 4.0**. The credit line the app
carries is unchanged, which is what the licence asks for.

Jost, Newsreader and JetBrains Mono are SIL Open Font License.

---

## Linking to a season

State lives in the URL fragment:

```
index.html#season=2022&tab=opponents
```

`season` is a season name, `2018` to `2026`, or `ALL`. `tab` is one of
`seasons`, `attack`, `squad`, `opponents`, `rivals`, `crowd`. The back button
works.

In the Omni build the same state is carried by `omni.setParams`, which has no
equivalent outside the sandbox. The application code is identical in both.

---

## Where the data comes from

The [American Soccer Analysis](https://app.americansocceranalysis.com) public
API, read at query time by MotherDuck with `read_json_auto`, shaped by seven
Omni SQL query views, and **frozen here on 16 August 2026**.

The Omni build still reads live. This one cannot, and says so in the footer
rather than implying otherwise. A page that called a third-party sports API from
the browser would break the first time that API changed its CORS policy,
rate-limited a shared origin, restructured a field, or went down.

| Query | Rows | Grain |
|---|---|---|
| Season Summary | 9 | season |
| Team Phases | 63 | season × action type |
| Opponent Record | 166 | season × opponent |
| League Context | 246 | season × club |
| Squad Season | 257 | season × player |
| Match Log | 300 | match |
| Player Actions | 1,410 | season × player × action type |

Columns keep their Omni field IDs, `<view>.<column>`, for example
`match_log.goals_for`, so any figure on screen traces back to a query without a
translation step. These are SQL query views, so there is no schema in the
prefix.

Every season is loaded at once and the app narrows in the browser, which is how
the Omni build worked too. Nothing is filtered server side.

Sanity check on the data: the nine seasons sum to **522 points**, which is the
figure the app shows for ALL.

---

## Rebuilding

```
python3 build_data.py    # seven CSVs in raw/ -> app-data.json
python3 build.py         # -> dist/index.html, standalone, and the Omni build
python3 verify_web.py    # serves dist/ over HTTP and checks it
```

`build_data.py` asserts that every one of the 98 fields the application reads is
present in the exports, so a re-export with a renamed column fails the build
rather than rendering blanks.

`verify_web.py` also takes an origin, which is how the deployed site gets
checked against its own bytes:

```
python3 verify_web.py https://astrodataus.github.io/lafc-season/
```

### Refreshing the snapshot

Re-export the seven queries from the Omni workbook as CSV with Row limit set to
**All possible results**, drop them in `raw/`, then:

```
python3 build_data.py && python3 build.py && python3 verify_web.py
```

Update the snapshot date in `build.py` (`SOURCE_NOTE`) so the footer keeps
telling the truth, then commit and push.
