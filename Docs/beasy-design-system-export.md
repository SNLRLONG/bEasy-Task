# bEasy Design System — Knowledge Export

Exported 2026-09-28 for review. Reply with corrections by line ID (e.g. "C-07: wrong, should be …").

## Where this came from

| Tag | What it means here |
|---|---|
| **[memory]** | My persistent memory for this project. **It's empty, so nothing in this export is tagged [memory].** |
| **[instructions]** | Three synced skills loaded as custom instructions: `suntel-design-system` (last verified 8 Aug 2026), `beasy-customer-app` (13 Aug 2026), `beasy-partner-app` (12 Aug 2026). These are Figma-side facts. |
| **[project file]** | Read from the Flutter code in `bEasy-app/` today (`lib/util/color_resources.dart`, `dimensions.dart`, `style.dart`, `get_font_family.dart`, `lib/theme/*`, `lib/view/custom/*`, `pubspec.yaml`). This is what is actually shipped. |
| **[past chat]** | I can't see any past chats from this session, so nothing is tagged [past chat]. |
| **[inferred]** | My own reading of the sources above. Treat as unverified. |

⚠️ **The Figma system ([instructions]) and the shipped Flutter code ([project file]) disagree in many places.** I've reported both side by side and haven't tried to reconcile them. Section Q lists the conflicts.

---

## C — Colours

### Brand
- **C-01** bEasy brand primary = `#1B4CFA`. [instructions] (sure)
- **C-02** bEasy gradient = `#1B4CFA` → `#102C90`, 2 stops, linear. Stored as a Figma **colour style**, because variables can't hold gradients. [instructions] (sure)
- **C-03** White text on `#1B4CFA` = 6.07:1, passes AA. `#171717` on `#1B4CFA` = 1.52:1, fails. On-brand text for bEasy is **white**. [instructions] (sure)
- **C-04** Local shim collection **`bEasy Brand (local shim)`** (in v2, must be recreated in v3): `bg-brand-primary` `#1B4CFA` · `bg-brand-pressed` `#102C90` · `bg-brand-soft` `#EEF3FF` · `text-brand-primary` `#1B4CFA` · `icon-brand-primary` `#1B4CFA`. [instructions] (sure)
- **C-05** Code: `ColorResources.appBarGradient` / `ColorResources.gradient` = `#102C90` → `#1B4CFA`, topLeft → bottomRight. The stop order is **reversed** from C-02. [project file] (sure)
- **C-06** Code: `AppColor.linearGradient` (text shader) = `rgb(27,76,250)` → `rgb(16,44,144)`, which is `#1B4CFA` → `#102C90`. [project file] (sure)
- **C-07** Code: `#163CC5` shows up as `ColorResources.violetColor`, `ColorResources.greyBold` (a misleading name: it's blue, not grey) and `AppColor.bEasyGradientStart`. It's the most-used brand colour in code (`greyBold` has 110 references). It is **not in the Figma system**. [project file] (sure)
- **C-08** Code: an `AppColor.bEasy*` ramp exists but is a **different blue** from Figma. Light `#EAEFF7` · LightHover `#DFE8F3` · LightActive `#BECFE5` · Normal `#2C63AC` · NormalHover `#234F8A` · NormalActive `#234F8A` · Dark `#214A81` · DarkHover `#1A3B67` · DarkActive `#142D4D` · Darker `#0F233C`. [project file] (sure)
- **C-09** Code: raw hex literals appear alongside the tokens: `0xFF1B4CFA` ×23, `0xFF102C90` ×23, `0xFF163CC5` ×24. [project file] (sure)
- **C-10** Code: `AppColor.suntel*` ramp (Suntel red): Light `#FCEAEB` · LightHover `#FADFE1` · LightActive `#F5BEC0` · Normal `#E02C35` · NormalHover `#CA2830` · NormalActive `#B3232A` · Dark `#A82128` · DarkHover `#861A20` · DarkActive `#651418` · Darker `#4E0F13`. [project file] (sure)
- **C-11** Code: `AppColor.bNow*` ramp is present in the bEasy app (Normal `#F8B625`, Light `#FEF8E9`, Dark `#BA891C`, Darker `#57400D`, plus the hover/active steps). [project file] (sure)
- **C-12** Other Suntel brand gradients, for reference: bNow `#FF8C42` → `#FDB01A` (41%) → `#FDB514` at 45°; YOK `#CA1C48` → `#800020`; bHR has no gradient. [instructions] (sure)
- **C-13** Rule: never use one product's brand colour in another. `#1B4CFA` is bEasy's, but the bNow Admin library uses it for link, focus and info. Links, focus rings and info states should come from the `Info` ramp. [instructions] (sure)
- **C-14** Rule: gradients below ~32px turn into a smudge, so use the flat solid there. Text on a gradient must pass at its worst stop. [instructions] (sure)

### Text greys
- **C-15** Figma semantic text tokens (light / dark): `text-gray-primary` Gray/900 / Gray/100 · `text-gray-secondary` Gray/700 / Gray/300 · `text-gray-tertiary` Gray/600 / Gray/400 · `text-gray-disable` Gray/200 / Gray/800 · `text-inverse` White / Black. I don't have the Gray ramp hex values. [instructions] (sure on mappings, no hex)
- **C-16** Code: `AppColor.content*` = Default `#1A1A1A` · Subtle `#3D3D3D` · Dim `#707070` · Placeholder `#A3A3A3` · Disabled `#BFBFBF` · DefaultInverse `#FFFFFF`. `contentDefault` (102 uses) and `contentSubtle` (84) are the most-used colours in code. [project file] (sure)
- **C-17** Code, legacy greys in `ColorResources`: `greyColorShade` `#918B8B` · `greyLight` `#D9D9D9` · `blackLight` `#696B6C` · `blackColor` `#000000` · `greyMedium` `#77BEED` (light blue despite the name). [project file] (sure)
- **C-18** Code: `CustomNoticeCard` body text uses `#404040`. [project file] (sure)

### Backgrounds
- **C-19** Code: `AppColor.background*` = Default `#FFFFFF` · GrayPrimary `#FAFAFA` (commented as `Background/bg-gray-primary`) · Alt `#F5F5F5` · Disabled `#F5F5F5` · Translucent `#FFFFFF` @ 24% (`0x3D`) · DefaultInverse `#1A1A1A` · AltInverse `#3D3D3D` · TransparencyInverse `#1A1A1A` @ 80% (`0xCC`). [project file] (sure)
- **C-20** Code, legacy: `ColorResources.whiteBackgroundColor` `#F8F8F8` · `blueGreyColor` `#F5F6F8` · `topUpColor` `#CEEDFF`. [project file] (sure)
- **C-21** Figma library background tokens used by bEasy: `bg-gray-primary`, `bg-gray-secondary`, `bg-white-primary`. I don't have hex values. [instructions] (sure on names)

### Borders
- **C-22** Code: `AppColor.outline*` = Softest `#E8E8E8` · Soft `#DBDBDB` · Default `#BFBFBF` · Bolt (sic) `#A3A3A3` · Boldest `#8C8C8C` · Disabled `#E8E8E8`. `outlineSoftest` is the most-used one (34). [project file] (sure)
- **C-23** Figma: `border-gray-secondary` is the library token bEasy uses. The Border group adds a Quaternary step. No hex. [instructions] (sure)

### Success / error / warning / info
- **C-24** Code: `AppColor.success*` Light `#E9F9F1` · LightHover `#DEF6EA` · LightActive `#BAECD3` · Normal `#20C272` · NormalHover `#1DAF67` · NormalActive `#1A9B5B` · Dark `#189256` · DarkHover `#137444` · DarkActive `#0E5733` · Darker `#0B4428`. [project file] (sure)
- **C-25** Code: `AppColor.alert*` (error) Light `#FEEBEC` · LightHover `#FDE1E2` · LightActive `#FBC2C3` · Normal `#F33A3C` · NormalHover `#DB3436` · NormalActive `#C22E30` · Dark `#B62C2D` · DarkHover `#922324` · DarkActive `#6D1A1B` · Darker `#551415`. [project file] (sure)
- **C-26** Code: `AppColor.warning*` Light `#FEF7EA` · LightHover `#FDF3E0` · LightActive `#FBE7BF` · Normal `#F1B12F` · NormalHover `#D99F2A` · NormalActive `#C18E26` · Dark `#B58523` · DarkHover `#916A1C` · DarkActive `#6C5015` · Darker `#543E10`. [project file] (sure)
- **C-27** Code: `AppColor.info*` Light `#EBEFFE` · LightHover `#E1E7FE` · LightActive `#C0CEFC` · Normal `#3460F5` · Normal2 `#0063F7` · NormalHover `#2F56DD` · NormalActive `#2A4DC4` · Dark `#2748B8` · DarkHover `#1F3A93` · DarkActive `#172B6E` · Darker `#122256`. [project file] (sure)
- **C-28** Code, legacy status colours: `ColorResources.successColor` `#27AE60` · `warningColor` `#E2B93B` · `errorColor` `#EB5757` · `redColor` `#E68C8C` · `orangeColor` `#EA5D1A`. [project file] (sure)
- **C-29** Code, snackbars: success `#27AE60`, info `#2196F3`, error `#FF5252`. [project file] (sure)
- **C-30** Code: `CustomNoticeCard` background `#FFF8E8` (warm amber-tinted). [project file] (sure)
- **C-31** Figma semantic: `text-success-primary` Success/500 · `text-warning-primary` Warning/500 · `text-error-primary` Error/500 · `text-info-primary` Info/500 (dark mode uses /400). No hex values. [instructions] (sure)
- **C-32** Rule: `status/*` colours are calibrated as **fills**. As text they fail contrast (success 2.34:1, warning 1.88:1 on white), so use the `-text` variants. [instructions] (sure)
- **C-33** Gap: there is no `Background/bg-success-primary` semantic token, only `Success/dark`, `Icon/icon-success-primary` and component-scoped variants. [instructions] (sure)

### Status chips
- **C-34** Customer app chips: green `Completed` / `Paid` · amber `Pending` · red `Cancelled` · light-blue `In Progress`. [instructions] (sure)
- **C-35** Admin tables (partner side): amber `Pending` · pink/red `Unresolved`, `Failed`, `Inactive`, `Deduction Pending` · green `Success`, `Resolved`, `Active`, `Completed` · blue `Open` · yellow `Under Review`. [instructions] (sure)
- **C-36** Map Tracking status dots: `On-Site` green, `Available` blue. [instructions] (sure)
- **C-37** I don't have the exact hex values for the chip fills or chip text. [—] (sure that I don't have them)

### Other code colours
- **C-38** Code: `ThemeData` `primaryColor` is `#E1BB17` (yellow) in light and `#27AE60` in dark. Neither is bEasy blue. [project file] (sure)
- **C-39** Code: `CustomSmallButton` default gradient `#FAE777` → `#FBC916`, text `#611515`. Grid yellows `#FBE47E` / `#FDBC23`. Facebook `#1877F2`. [project file] (sure)

---

## T — Typography

- **T-01** Figma system fonts: **Inter** (desktop/web) · **SF Pro** (iOS) · **Roboto** (Android). [instructions] (sure)
- **T-02** bEasy v3 Figma uses **SF Pro** Regular / Medium / Semibold. [instructions] (sure)
- **T-03** Weights: Regular 400 · Medium 500 · Semibold 600 · Bold 700. [instructions] (sure)
- **T-04** Type scale as size/line-height: H1 64/82 (letter-spacing 3px) · H2 48/64 · H3 40/54 · H4 32/42 · H5 24/32 · H6 20/28 · H7 18/26 · H8 16/24 · Body 1 18/28 · Body 2 16/24 · Body 3 14/20 · Body 4 12/16 · Body 5 10/12. Letter-spacing is 0% except H1. [instructions] (sure)
- **T-05** Semantic group `metadata`: `metadata 01` 12/16 · `metadata 02` 10/12, each in regular / medium / bold. Use `metadata 01` for metadata even though `Body 4` has the same metrics. [instructions] (sure)
- **T-06** These line heights are canonical (decided 8 Aug 2026). Product files that differ should adopt them. [instructions] (sure)
- **T-07** Minimum size (approved 8 Aug 2026) is **12px**. 10px is **mobile-only and rare**, and only for text where nothing happens if nobody reads it. 10px is never allowed for helper text, errors or validation, status labels, or timestamps the user acts on. [instructions] (sure)
- **T-08** Pending: a 13/18 `Body 3.5` step (currently only in bNow Admin), approved in principle but not built. [instructions] (sure)
- **T-09** Code, English: `localizedTextStyle()` / `defualtTextStyle()` (sic) use **Inter via `google_fonts`**. [project file] (sure)
- **T-10** Code, Khmer: `GoogleFonts.kantumruyPro`, and the `fontSize` is **hard-coded to 15** no matter what the caller passes. [project file] (sure)
- **T-11** Code, `zh_cn` / `zh_tw` / `vi`: **MiSans** (bundled asset). [project file] (sure)
- **T-12** Code: `getFontFamily()` returns **Poppins** for en, KantumruyPro for km and MiSans for zh. This conflicts with T-09. [project file] (sure)
- **T-13** Code: `ThemeData.fontFamily` = **Mulish** in both light and dark themes, but no Mulish asset is declared in `pubspec.yaml`. [project file] (sure)
- **T-14** Code: in `pubspec.yaml`, the `Inter` family entry points at **ComicNeue-*.ttf** files. [project file] (sure)
- **T-15** Code: `Dimensions` font sizes on phones: ExtraSmall 10 · Small 12 · Default 16 · Large 16 · ExtraLarge 18 · OverLarge 24. Tablet and ≥1300px widths are larger. [project file] (sure)
- **T-16** Code: primary button label is 18 / w300. Get-Started button label is 13 / w300. [project file] (sure)

---

## S — Spacing & layout

- **S-01** Base unit 4px. Token name = multiplier: `space/1` 4 · `/2` 8 · `/3` 12 · `/4` 16 · `/5` 20 · `/6` 24 · `/8` 32 · `/10` 40 · `/12` 48 · `/16` 64. Not included, add on demand: `space/0.5` (2), `space/20` (80). [instructions] (sure)
- **S-02** Semantic, mobile / desktop: `gap/within` 12/12 · `gap/sibling` 16/24 · `gap/section` 40/48 · `padding/screen` 16/32 · `padding/card` 16/20 · `padding/page-top` 24/64. [instructions] (sure)
- **S-03** Invariant: `gap/section` ≥ 2 × `gap/sibling`. [instructions] (sure)
- **S-04** Observed in bEasy mobile Figma: 16 screen margin · ~12 within components · 40 between major sections. [instructions] (sure)
- **S-05** Screen width: mobile frames are **428px** (iPhone 13 Pro Max). Breakpoints are 1440 / 1024 / 428 only. Grid columns and gutters aren't defined. [instructions] (sure)
- **S-06** Code, `Dimensions` padding: 2 · 6 · 10 · 15 (default) · 16 (`homePagePadding`) · 17 · 20 · 25 · 30 · 50. Margin: 5 · 10 · 15 · 20 · 25 · 30 · 50. **Not on the 4px grid.** [project file] (sure)
- **S-07** Sizing tokens: `size/row-comfortable` 44 · `size/row-dense` 36 · `size/touch-target` 44 · `size/bottom-nav` 92 · `size/sidebar` 240 (desktop). [instructions] (sure)
- **S-08** Code: icon sizes 20 / 24 / 28 / 38 on phone. Primary button height 45. Get-Started and icon buttons 50. Small button 48. [project file] (sure)

---

## R — Shape & elevation

- **R-01** Radius primitives: `radius/xs` 4 · `sm` 8 · `md` 12 · `lg` 16 · `full` 9999. [instructions] (sure)
- **R-02** Semantic radius: `radius/input` 12 on mobile (8 on desktop) · `radius/button` full · `radius/button-square` 8 · `radius/badge` full · `radius/card` 12 · `radius/modal` 16 (modals, sheets, dialogs). [instructions] (sure)
- **R-03** Rule: a container's radius must be larger than anything nested inside it. [instructions] (sure)
- **R-04** Product files aren't consistent on radius. Migrate to the tokens rather than copying what's on screen. [instructions] (sure)
- **R-05** Code, radius frequency: 8 (×81) · 30 (×50, pill buttons) · 16 (×20) · 4 (×18) · 6 (×16) · 50 (×8) · 56 / 48 (×6 each) · 12 (×6) · 40 (×5) · 20 / 2 (×4 each). `Dimensions.borderRaduis` (sic) = 4. [project file] (sure)
- **R-06** Code: text fields use radius 6 with a `Colors.grey.shade300` border when filled. The Figma spec is 12 (R-02). [project file] (sure)
- **R-07** Code: snackbar radius 12. Notice card radius 16. [project file] (sure)
- **R-08** Mobile elevation: `level 1` 0/2, blur 12, `#1C1C1C` 10% · `level 2` 0/8, blur 24, `#1C1C1C` 10% · `reverse` 0/-2, blur 12, `#000000` 10% · `glass` (Glass: light −45° at 80%, refraction 100, depth 16, frost 12; plus drop shadow 0/8, blur 40, `#000000` 12%), used by exception and never on data-dense surfaces. [instructions] (sure)
- **R-09** Web elevation: `level 1` 0/4 b9 `#161A1D` 5% · `level 2` 0/5 b10 12% · `level 3` 0/8 b10 16% · `reverse` 0/-2 b12 `#1C1C1C` 10% (colour inconsistency is open). [instructions] (sure)
- **R-10** Code: `AppColor.boxShadow` = 0/**1**, blur 12, `rgba(28,28,28,0.10)`. Close to mobile `level 1` but Y is 1, not 2. [project file] (sure)
- **R-11** Code: snackbar shadow 1/1, blur 5, `grey.shade200`. `kDefaultBoxShadow` 0/8, blur 30, spread 1, `black26`. [project file] (sure)
- **R-12** Dividers: nothing specific in my sources beyond the outline colours in C-22. [—]

---

## K — Tokens & variables

- **K-01** Figma library: **"Design System"** file `jZcf2OMkMyi5QE2TfefxiF`, team `1530915245380204732`. [instructions] (sure)
- **K-02** Architecture: Primitive → Semantic (light + dark) → Applied. Components reference semantic tokens only. [instructions] (sure)
- **K-03** Primitive collections: Brand ramps per product (`bNow`, `bHR`, `bEasy`; YOK pending), each 50–950 · `Gray` · Base `White`/`Black` at 100/60/40/5% · Status `Success`, `Warning`, `Error`, `Info` · 18 secondary hues (Red … Slate). [instructions] (sure)
- **K-04** Semantic groups: **Text**, **Icon**, **Border** (with Quaternary), **Background**, **Utilities** (Level 0.5–9.5 per hue). [instructions] (sure)
- **K-05** Naming is kebab-case and property-first (`text-brand-primary`). Slash paths are only the conceptual grouping. [instructions] (sure)
- **K-06** 🔴 **Library brand tokens resolve to bNow amber, not bEasy blue** (`bg-brand-primary`, `bg-brand-secondary`, `text-brand-primary`, `icon-brand-primary`). The cause is that brand modes on the primitive collection haven't been built ("status: not started"). [instructions] (sure)
- **K-07** Workaround: the **local** `bEasy Brand (local shim)` collection (C-04). Delete it once library brand modes land. Bind non-brand tokens (gray, text, success, error, border) from the library. [instructions] (sure)
- **K-08** Library variable keys, truncated as I have them: `bg-brand-primary` 4f0dce9d… · `bg-brand-secondary` 84dc7485… · `bg-gray-primary` e015fbd3… · `bg-gray-secondary` d8998824… · `bg-white-primary` 475555d0… · `text-gray-primary` 0b0b07c8… · `text-gray-secondary` 0eb628dd… · `text-gray-tertiary` ed109796… · `text-gray-disable` 618b3f63… · `text-inverse` bab7af30… · `text-brand-primary` 0c2100a9… · `text-error-primary` a72a100c… · `text-success-primary` d83b1f95… · `icon-brand-primary` af36faf1… · `icon-gray-secondary` 2aad1af7… · `icon-success-primary` 70d81a7e… · `border-gray-secondary` cbad9c1b… · `Success/dark` f2d0ab8b…. [instructions] (sure, truncated)
- **K-09** Proposed, not built: a `text-brand-on-fill` token (bEasy → White/100%). [instructions] (sure)
- **K-10** bEasy v3 file has **no local variables and no text, effect or paint styles**. [instructions] (sure)
- **K-11** Figma variable scope enum uses `STROKE_COLOR`, not `STROKE`. [instructions] (sure)
- **K-12** Code has no token layer that maps to Figma names. Colours live in two classes: legacy `ColorResources` and a newer `AppColor` (Light/Normal/Dark ramps plus content/background/outline). `AppColor.backgroundGrayPrimary` is the only one commented with a Figma name. [project file] (sure)
- **K-13** Spacing, radius and sizing tokens (S-01, R-01, S-07) exist as a spec. I don't know whether they exist as Figma variables. [instructions] (unsure)

---

## P — Components

- **P-01** Button matrix convention: 4 sizes × 4 states (default, hover, pressed, **disabled**) · circular icon-only variant at every size · variants solid, soft/tinted, outline, ghost/subtle and link, each in brand and neutral · leading and trailing icon slots. Library Button node `411:2647`. [instructions] (sure)
- **P-02** bEasy primary button: solid `#1B4CFA`, full-width, pill. [instructions] (sure)
- **P-03** Code: `CustomButtonWidget.buildButtonClick` is 45 high, radius 30, **gradient** fill (`appBarGradient`), white 18 / w300 label. The inactive state is a white fill with a `#163CC5` label. [project file] (sure)
- **P-04** Code: `buildButtonGetStart` / `buildButtonWithIcon` are 50 high, radius 8, solid `#163CC5`. The inactive state is a white fill with a grey 1px border. [project file] (sure)
- **P-05** Library components also include: Avatar, Badges, Check Box, Dropdown, Input, Tags, Toggles. I don't have node IDs apart from Button. [instructions] (sure)
- **P-06** v3 component sets: `radio button` `43:370`, `phone number` `43:1571` (inside `43:335`). Phone input states `default` `49:477` · `idle` `57:851` · `active` `57:1063`. [instructions] (sure)
- **P-07** OTP input: 6 digits, with states default / loading / resend (`4m56s` countdown) / filled / error. [instructions] (sure)
- **P-08** Code: `CustomTextFieldWidget`: 16 label, 14 grey hint, `black45` cursor, radius 6. [project file] (sure)
- **P-09** Chips: see C-34 / C-35. Badges are fully round (`radius/badge`). [instructions] (sure)
- **P-10** Overlay sheet **`Scope of Work`** (v3 `6:611`): tasks grouped by room (Kitchen, Bathroom, Living room/Communal area) as bullet lines. Reuse it for the completion walkthrough. [instructions] (sure)
- **P-11** v3 screen anatomy: `super-header` / `service type` / `address` / `bottom nav` / `overlay` blocks at 428px. [instructions] (sure)
- **P-12** Bottom nav: **Home · Booking · Notifications · Account**. `size/bottom-nav` 92. [instructions] (sure)
- **P-13** Headers use the blue gradient. [instructions] (sure)
- **P-14** Date/time picker: code has `custom_calendart.dart`, `custom_schedule_date_selector.dart`, `custom_schedule_time_picker.dart`, `custom_instant_time_selector.dart`, `custom_unavailable_schedule_sheet.dart`. I haven't read their visual specs, and there is nothing in the Figma sources. [project file] (sure they exist; no spec)
- **P-15** Dialogs and sheets in code: `showModalBottomSheet` ×41, `showDialog` ×15, `Get.dialog` ×4. Wrappers include `custom_show_dialog.dart`, `custom_show_modal_pop_up.dart`, `custom_payment_sheet.dart`, `custom_task_info_sheet_v2.dart`, `custom_equipment_sheet_v2.dart`, `custom_category_selection_alert_sheet.dart`. [project file] (sure)
- **P-16** Snackbar: 50 high, radius 12, 16 / w600 white text; success / info / error colours per C-29. [project file] (sure)
- **P-17** Notice card: `#FFF8E8` background, radius 16, black 14 / w600 title, `#404040` 14 / w500 body. [project file] (sure)
- **P-18** Cards: `radius/card` 12, `padding/card` 16 on mobile. I have no bEasy-specific card spec. [instructions] (sure)
- **P-19** Customer order screens with **Assigned Partners** (avatar row + count + trust banner) live in the **Partner App** file (`603:6129`, `596:5266`), not in v2 or v3. [instructions] (sure)

---

## U — UX patterns & rules

- **U-01** Sheets vs full screens: `Scope of Work`, cross-sale and the guest $5-credit offer are overlays or modals. I have no stated general rule. [instructions] (sure on the examples, no rule)
- **U-02** Checkout price breakdown order: Service · Discount · Service Fee · Transport Fee · Subtotal · VAT 10% · Total. Cross-sell discount is a separate `Cross Sell` line. [instructions] (sure)
- **U-03** Payment methods: `bWallet`, `ABA KHQR`, `Credit/Debit Card`. **`Pay After Service` was removed (Aug 2026).** [instructions] (sure)
- **U-04** Sign-off framing: the customer paid at checkout, so completion and sign-off screens don't use money language. [instructions] (sure)
- **U-05** Completion: an inverted checklist where everything is assumed fine and the customer taps only what's wrong. Three outcomes: confirm · **ask for a redo** (prominent) · report an issue (quiet). There is no auto-complete; `Completed (admin)` is a distinct state. The mismatch dialog is framed as a reconciliation, not a warning. [instructions] (sure)
- **U-06** Coupons: non-matching coupons are shown **disabled, not hidden**. Threshold hints read like "Add $4.28 more". `x N left`. Coupons aren't valid on add-ons. [instructions] (sure)
- **U-07** Credit rules: bWallet cash never expires and can be withdrawn. bCombo bonus (buy $50 get $100) can't be withdrawn, and **buying it forfeits the bFamily coupon**. bCombo doesn't apply to top-ups. [instructions] (sure)
- **U-08** Refunds and fees: a 20% reschedule / 50% cancellation fee is an incoming requirement. No cancellation flow exists and reschedule is free today. The recommendation is to tier fees by notice and add a grace window. Closing early has no refund or penalty but must be recorded. [instructions] (sure)
- **U-09** Colour meaning: red = error, destructive, cancelled. Destructive actions are **red text on white** (partner/admin). Amber = pending, warning, business-rule notes. Blue = in progress, open, info, available. Green = completed, paid, success. **Links, focus and info should come from the `Info` ramp, not the brand ramp.** [instructions] (sure)
- **U-10** Never use amber-family text on white (the "amber rule" is referenced but not defined in my sources). [instructions] (unsure of the exact wording)
- **U-11** Disabled: a missing disabled state counts as an incomplete component. Coupons use a disabled state instead of hiding. `text-gray-disable` → Gray/200. [instructions] (sure)
- **U-12** Empty states: known gaps with no designed state: empty cart, booking-history empty. The partner app has `home/online/empty` and `message_empty`. [instructions] (sure)
- **U-13** Error states: in v2 they're red-bordered "Error State" containers next to the screen. OTP has an error state. The phone field has `invalid number`. [instructions] (sure)
- **U-14** Copy tone: examples only. "All cleaners are verified, trained, rated by us" · review scale "Awful Experience → Loved it!" · "What could be improved?" · "One Item at a Time — your cart can only hold one item". There's no written tone guide. [instructions] (sure on examples)
- **U-15** Auth is Telegram-first (Telegram → Apple → phone). Android doesn't show Apple. There's a "How did you get to know us?" step after sign-up. [instructions] (sure)
- **U-16** Guest homepage shows `Sign in / Sign up` plus a $5-credits modal. [instructions] (sure)
- **U-17** Cross-sale is shown once per checkout session. It hides categories already in the cart (except Upholstery) and adds to the cart immediately. [instructions] (sure)

---

## F — Figma references

- **F-01** Design System library: `jZcf2OMkMyi5QE2TfefxiF`. Colors `145:2` · Typography `1:3` · Button `411:2647`. [instructions] (sure)
- **F-02** **bEasy - v3 (current)**: `Q6phvltGCcVJK41XlOdE7c`. Pages: `📁 Ready For Dev / ↳ UI` `1:6` (empty) · `📁 Design In-Progress / ↳ UI` `1:7` · `Page 8` `142:789`. [instructions] (sure)
- **F-03** v3 frames: `scope of work` `6:611` (duplicates `36:9875`, `36:10222`, `36:10569`) · general cleaning `6:197` · deep cleaning `19:89` · upholstery `31:269` · pest `31:755` · ac `31:1324` · washing machine `33:1931` · login/phone `36:9745`, `43:675` · quotation form `68:1355` (894×1800). [instructions] (sure)
- **F-04** v3 task lists: general `67:1157` · deep `67:789` · upholstery `67:1196` · pest `67:1268` · ac `67:1006` · washing machine `67:863`. [instructions] (sure)
- **F-05** **bEasy - v2 (legacy)**: `OwAMYCTuEnZu6a9yd4blFA`. Authentication `2957:7763` · Homepage `265:10393` · Top-Up/bWallet `1733:7353` · Service details `75:5575` · Booking history `101:14411` · Cross-sale `3449:7225` · Coupon `3434:1791` · Rebook `3084:6769` · Remove from cart `3506:8042` · Notifications `2700:8986` · Profile `981:3681`. [instructions] (sure)
- **F-06** **Partner App**: `l3NZavFOdNyuZ0RhmZched`. Page `0:1` · mobile root `2:2` · admin root `2:5`. Mobile sections: auth `14:2757`, home `14:2760`, job details `26:1064`, profile `71:1447`, order details `596:5266`. Admin: map tracking `369:2069`, payout `377:4877`, partners `432:1977`, chat `521:565`, ticket `682:7970`. Customer order with partners `603:6129`. [instructions] (sure)
- **F-07** The bEasy Admin Portal is a separate Figma file. I don't have its key. [instructions] (sure)
- **F-08** Tooling notes: `get_metadata` fails on large sections, so use `get_screenshot` and crop. Page nodes return an empty canvas. Styles and variables are only readable from the panel. [instructions] (sure)

---

## X — Other developer-relevant facts

- **X-01** Flutter code and Figma have drifted: two colour classes, a non-Figma blue `#163CC5` as the main UI blue, a stale `bEasy*` ramp (`#2C63AC`), off-grid spacing (15/17/25), and a primary button with a gradient where Figma has solid. [project file] + [inferred] (sure on facts, inferred on "drift")
- **X-02** Languages in the app: English, Khmer, Vietnamese, Chinese Simplified/Traditional. Fonts per locale are in T-09 to T-11. [instructions] + [project file] (sure)
- **X-03** Pchum Ben holiday pause is hard-coded to 2025 dates. [instructions] (sure)
- **X-04** Page transitions: `ZoomPageTransitionsBuilder` on all platforms. [project file] (sure)
- **X-05** Dark theme exists in code (`lib/theme/dark_theme.dart`) but is minimal. The Figma system treats dark mode as first-class. [project file] + [instructions] (sure)
- **X-06** Equipment image library in v3: ~30 frames at 90×90. [instructions] (sure)

---

## Q — Unsure or contradictory

- **Q-01** **Gradient direction.** Figma says `#1B4CFA` → `#102C90`. Code `appBarGradient` runs `#102C90` → `#1B4CFA` (topLeft → bottomRight). Which is correct?
- **Q-02** **Which blue is primary in UI?** Figma uses `#1B4CFA`. Code uses `#163CC5` most (`greyBold` / `violetColor` / `bEasyGradientStart`). Is `#163CC5` intentional?
- **Q-03** **`AppColor.bEasy*` ramp** (`#2C63AC` normal) doesn't match either blue. Is it dead code or an older brand?
- **Q-04** **Primary button**: Figma says solid, code uses a gradient. Also radius 30 (primary) vs 8 (Get-Started) vs Figma full.
- **Q-05** **Font**: Figma says SF Pro (iOS) / Roboto (Android). Code uses Inter (en) via google_fonts, `getFontFamily()` says Poppins, `ThemeData` says Mulish, and pubspec maps "Inter" to ComicNeue files. Which is the intended app font?
- **Q-06** Khmer text is forced to 15px no matter what size is requested. Is that intentional?
- **Q-07** Input radius: Figma says 12 on mobile, code uses 6.
- **Q-08** Shadow: code `boxShadow` Y=1 vs mobile `level 1` Y=2.
- **Q-09** `ThemeData.primaryColor` is yellow `#E1BB17` (light) / green `#27AE60` (dark). Leftover from a template?
- **Q-10** Two parallel status palettes in code (`ColorResources.successColor` `#27AE60` vs `AppColor.successNormal` `#20C272`; `errorColor` `#EB5757` vs `alertNormal` `#F33A3C`). Which one wins?
- **Q-11** `bg-brand-soft` `#EEF3FF` (shim) vs code `AppColor.infoLight` `#EBEFFE` / `bEasyLight` `#EAEFF7`. Which is the soft brand tint?
- **Q-12** Library brand tokens render bNow amber for bEasy (K-06). Is it still true that brand modes aren't built?
- **Q-13** Cart: "one item at a time" vs the cart-removal flow showing two services.
- **Q-14** Credit shown in both dollars and "Points", with no conversion.
- **Q-15** Canonical `scope of work` frame: `6:611` or one of its three duplicates?
- **Q-16** Exact hex values for chip fills and text, the Gray ramp, `Brand/500` for bEasy, and the `Info` ramp in Figma. I don't have any of these.
- **Q-17** Whether spacing, radius and sizing tokens exist as real Figma variables or only as a written spec.
- **Q-18** Sheets vs full-screen rule: I don't have a stated rule, only examples.
- **Q-19** Copy tone guide: none in my sources.
- **Q-20** "bEasy is a home-cleaning booking app": my sources describe it more broadly as home services (cleaning, AC, upholstery, pest, washing machine, electrical). Phnom Penh is only mentioned for the Map Tracking screen.
