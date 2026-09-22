# Venus University character workflow

`VenusUniversityCharacter.workflow.json` is a ComfyUI workflow based off the workflows I use to generate characters in my game Venus University: https://venus-dev.itch.io/venus-university. you type an appearance, up to four outfits, a pose and a seed, then press Queue once. Out come a base frame and seven expression sprites with transparent backgrounds for every outfit, plus an optional set of eight NSFW CGs once you turn that part on.

## What you get

One queue press writes, for each outfit box you filled in:

```
ComfyUI/output/VenusUniversity/<outfit>/neutral.base_00001_.png   opaque base frame
ComfyUI/output/VenusUniversity/<outfit>/neutral_00001_.png        transparent sprite
ComfyUI/output/VenusUniversity/<outfit>/happy_00001_.png
ComfyUI/output/VenusUniversity/<outfit>/sad_00001_.png
ComfyUI/output/VenusUniversity/<outfit>/angry_00001_.png
ComfyUI/output/VenusUniversity/<outfit>/surprised_00001_.png
ComfyUI/output/VenusUniversity/<outfit>/embarrassed_00001_.png
ComfyUI/output/VenusUniversity/<outfit>/aroused_00001_.png
```

`<outfit>` is whatever name you wrote at the start of that outfit line. Eight files per outfit: one base frame plus seven sprites.

If you switch the NSFW CG group on, you also get:

```
ComfyUI/output/VenusUniversity/cg/nude_foreplay_00001_.png
ComfyUI/output/VenusUniversity/cg/nude_foreplay_after_00001_.png
ComfyUI/output/VenusUniversity/cg/sex_00001_.png
ComfyUI/output/VenusUniversity/cg/sex_after_00001_.png
ComfyUI/output/VenusUniversity/cg/handjob_00001_.png
ComfyUI/output/VenusUniversity/cg/handjob_after_00001_.png
ComfyUI/output/VenusUniversity/cg/fellatio_00001_.png
ComfyUI/output/VenusUniversity/cg/fellatio_after_00001_.png
```

## Requirements

ComfyUI 0.34 or newer.

Three custom node packs:

- [ComfyUI-Impact-Pack](https://github.com/ltdrdata/ComfyUI-Impact-Pack) (Face Detailer, Make List)
- [ComfyUI-Impact-Subpack](https://github.com/ltdrdata/ComfyUI-Impact-Subpack) (the Ultralytics detector provider)
- [ComfyUI-Inspyrenet-Rembg](https://github.com/john-mnz/ComfyUI-Inspyrenet-Rembg) (background removal)

Six model files. The workflow selects these by exact filename, so rename where the table says to.

| Download page | File you get | Rename to | Put in |
| --- | --- | --- | --- |
| [Nova Anime XL](https://civitai.com/models/376130/nova-anime-xl), version IL v19.0 | `NovaAnimeILV190.safetensors` | `novaAnimeXL_ilV190.safetensors` | `models/checkpoints` |
| [USNR STYLE](https://civitai.com/models/176554/usnr-style), version USNR_STYLE_ILL_V1.0 | the LoRA file for that version | `usnrStyle.safetensors` | `models/loras` |
| [Illustrious-XL ControlNet Openpose](https://civitai.com/models/1359846/illustrious-xl-controlnet-openpose), version v1.0 | the ControlNet file for that version | `illustriousOpenpose.safetensors` | `models/controlnet` |
| [Real-ESRGAN v0.2.2.4](https://github.com/xinntao/Real-ESRGAN/releases/tag/v0.2.2.4) | `RealESRGAN_x4plus_anime_6B.pth` | keep the name | `models/upscale_models` |
| [Bingsu/adetailer](https://huggingface.co/Bingsu/adetailer) | `hand_yolov9c.pt` | keep the name | `models/ultralytics/bbox` |
| [Anzhc/Anzhcs_YOLOs](https://huggingface.co/Anzhc/Anzhcs_YOLOs) | `Anzhc Face seg 640 v2 y8n.pt` | `Anzhc%20Face%20seg%20640%20v2%20y8n.pt` | `models/ultralytics/segm` |

That last one is not a typo. Save the face segmentation model with literal `%20` characters where the spaces were, because that is the name the workflow selects. If your browser saves it with real spaces, rename it by hand.

### GPU

This is an SDXL-class checkpoint plus a ControlNet, at 832x1216 with a hires pass and two detailer passes. 12 GB of VRAM is comfortable. 8 GB works if you run fewer outfits per queue press and use the speed tips near the end of this file.

## Setup

1. Install the three node packs and restart ComfyUI.
2. Put the six model files where the table says.
3. Drop `VenusUniversityCharacter.workflow.json` onto the ComfyUI canvas.
4. If any node comes up red, the pack that owns it is missing. Install it and reload the page.
5. If a node loads but one of its model dropdowns is red or empty, the file is in the wrong folder or has the wrong name. Check it against the table.
6. Pick a pose, then load its skeleton: press the upload button on the **Pose skeleton** node and choose the matching PNG from the `skeletons/` folder next to this file. See the pose table below for which file goes with which pose.

The workflow opens on a START HERE note. Every group on the canvas carries a note explaining what it does.

## Using it

Everything you edit is in the group on the left. The workflow ships prefilled with a real character from the game, so you can press Queue straight away to see what it produces, then start replacing text.

| Box | What goes in it |
| --- | --- |
| **Appearance** | Her body, hair, eyes, head accessories and makeup, as comma separated booru tags. One box, used by every image. See the appearance guide below. |
| **Outfit 1** to **Outfit 4** | One outfit per box, written `name: tag, tag, tag`. The name before the colon becomes the output folder. Outfit 4 ships empty and muted. |
| **Pose tags** | The tags for the pose you picked, for example `hand_on_own_hip, arm_at_side`. |
| **Pose skeleton** | The OpenPose PNG for that same pose, uploaded from `skeletons/`. The tags and the skeleton must describe the same pose. |
| **Character seed** | One number that fixes who she is. Leave the control on `fixed`. |
| **Negative** | What to keep out of every image. The game uses `worst_quality, bad_quality, lowres, holding, backlighting, cum` and appends any per character negatives after it. |
| **Expression 1: neutral** to **Expression 7: aroused** | Seven boxes, one per expression, holding the face tags for that emotion. See the expression guide below. |

To skip an outfit, select its box and press Ctrl+M to mute it. The run then produces one folder fewer and takes proportionally less time. Unmute the same way. A muted box does not need to be emptied.

Then press Queue once. That is the whole run: the outfit boxes fan out into a list, and every stage after them runs once per outfit.

On an RTX 4090 the prefilled three-outfit run, 24 images, takes about four and a half minutes, so budget roughly ten seconds an image there and proportionally more on a smaller card. Files appear under `ComfyUI/output/VenusUniversity/` as they finish, so you can watch the folders fill.

### The prefilled example

This is April Valentine, who ships with the game. Her boxes read:

```
Appearance
light_pink_hair, light_blue_eyes, long_hair, curly_hair, wolf_cut, parted_bangs,
sidelocks, gold_hoop_earrings, silver_hairclips, eyeshadow, eyelashes, big_breasts

Outfit 1
everyday: pink_cropped_top, white_faux_fur_jacket, black_denim_miniskirt, black_lace_thighhighs, pink_platform_heels, gold_chain_belt, cleavage, navel, thighs, bare_arms

Outfit 2
pe: white_sports_bra, pink_short_shorts, white_sneakers, pink_visors, cleavage, navel, bare_legs, bare_arms

Outfit 3
swim: pink_micro_bikini, gold_belly_chain, cleavage, navel, bare_legs, bare_arms, bare_shoulders

Pose tags
hand_on_own_hip, arm_at_side

Pose skeleton
hand_on_hip.png

Character seed
1810879740387086

Negative
worst_quality, bad_quality, lowres, holding, backlighting, cum

neutral       (smug:0.7), light_smile, looking_at_viewer
happy         happy, grin, looking_at_viewer
sad           (sad:0.8), (frown:0.7), tears
angry         (angry:0.7), furrowed_brow, clenched_teeth
surprised     (surprised:0.8), open_mouth, (wide-eyed:0.8)
embarrassed   (embarrassed:0.8), parted_lips, blush
aroused       (smug:0.8), parted_lips, (half-closed_eyes:0.8), blush
```

That run writes `everyday/`, `pe/` and `swim/`, eight PNGs each.

## Appearance guide

The game writes a character's appearance from a fixed set of categories. You do not have to obey it, but the game writes its own cast from these categories, so a character written this way sits next to the shipped characters without looking out of place.

Colours take an optional shade prefix, `light_` or `dark_`, and a suffix, so the tag reads `{shade}_{color}_hair` or `{shade}_{color}_eyes`: `light_pink_hair`, `dark_black_hair`, `light_blue_eyes`, `dark_red_eyes`. Plain `blue_hair` works too.

| Category | Pick | Values |
| --- | --- | --- |
| Hair colour | 1 | `aqua`, `black`, `blonde`, `blue`, `brown`, `green`, `grey`, `orange`, `pink`, `purple`, `red`, `white` |
| Eye colour | 1 | `aqua`, `black`, `blue`, `brown`, `green`, `grey`, `orange`, `pink`, `purple`, `red`, `yellow` |
| Shade | 0 or 1 per colour | `light`, `dark` |
| Hair length | 1, as `{length}_hair` | `very_short`, `short`, `medium`, `long` |
| Hair texture | 1, as `{texture}_hair` | `straight`, `wavy`, `curly`, `messy` |
| Hair style | 1 or more | `bob_cut`, `hime_cut`, `pixie_cut`, `wolf_cut`, `inverted_bob`, `high_ponytail`, `low_ponytail`, `side_ponytail`, `twintails`, `low_twintails`, `single_braid`, `twin_braids`, `double_bun`, `single_hair_bun`, `one_side_up`, `two_side_up`, `drill_hair` |
| Bangs | 1 or more | `blunt_bangs`, `swept_bangs`, `parted_bangs`, `crossed_bangs`, `hair_between_eyes`, `hair_over_one_eye`, `sidelocks`, `long sidelocks` |
| Head accessories | any, each naming a colour or a material | `hairband`, `headband`, `hair_ribbon`, `hair_bow`, `hairclip`, `hair_flower`, `hair_ornament`, `glasses`, `earrings`, `ear_piercing` |
| Makeup | 0 to 2, with a colour on eyeshadow | `eyeshadow`, `eyelashes` |
| Breast size | 1, as `{size}_breasts`, omit for medium | `small`, `big` |
| Skin | 1, omit for fair | `tan`, `dark-skinned_female`, `freckles` |

Hair can be `blonde` or `white` but not `yellow`; eyes can be `yellow` but not `blonde` or `white`. Everything worn above the neck goes in the appearance box rather than in an outfit: earrings, glasses, hairclips.

Two more lines from the shipped cast:

```
Ana Fina
light_brown_hair, light_green_eyes, curly_hair, double_bun, crossed_bangs,
round_glasses, eyelashes, small_breasts, green_hair_ribbon, long_hair

Roxy Villeneuve
dark_black_hair, light_pink_eyes, short_hair, messy_hair, wolf_cut, crossed_bangs,
silver ear_piercing, black spiked earrings, black eyeshadow, eyelashes, small_breasts
```

As Roxy shows, an accessory tag is free text once it carries a colour or a material, and the shipped characters spell them both with underscores and with spaces.

## Outfit guide

An outfit line is a name, a colon, then booru tags:

```
everyday: pink_cropped_top, white_faux_fur_jacket, black_denim_miniskirt, black_lace_thighhighs, pink_platform_heels, gold_chain_belt, cleavage, navel, thighs, bare_arms
```

The rules the game writes its own outfits under:

- **Every clothing tag names at least one colour or material.** `white_sports_bra`, not `sports_bra`. This is most of what keeps her wearing the same clothes from one sprite to the next.
- **Name the skin that is showing**, using these tags: `navel`, `bare_legs`, `thighs`, `bare_arms`, `bare_shoulders`, `cleavage`, `clothing_cutout`. They pin the silhouette down so it does not drift between expressions.
- `bare_legs` is for fully bare legs. Use `thighs` for the strip of skin above thighhighs or socks.
- **No held items.** No handbags, totes, umbrellas or phones; her hands are empty. `holding` sits in the negative prompt for that reason.
- **Nothing on her head or ears.** No hats, no animal ears. Earrings, hairclips and glasses belong in the appearance box.

The game gives each character three outfits, which is why three boxes ship filled in. One of each from the shipped cast:

```
Outfit 1, everyday wear (Ginny Hadron)
everyday: bare_shoulders, gold_necklace, black_leather_boots, dark_purple_off-shoulder_sweater, black_tank_top, black_skinny_jeans

Outfit 2, PE kit (Ana Fina)
pe: pastel_pink_crop_top, black_booty_shorts, bare_legs, navel, bare_arms, white_running_shoes

Outfit 3, swimsuit (Roxy Villeneuve)
swim: black string halter bikini top, cleavage, bare_arms, bare_shoulders, navel, black cheeky side-tie bikini bottoms, bare_legs, silver body chain
```

**Swimsuit tip.** When the game renders a swimsuit it adds four extra negatives, because legwear from the everyday outfit tends to survive the change of clothes: `pantyhose, thighhighs, pants, gloves`. This workflow has one shared negative box, so if your swim frames come back wearing leftovers, add those four tags to the Negative box and rerun with only the swim box unmuted.

## Poses

Pick one pose, then set both the **Pose tags** box and the **Pose skeleton** image to it. The two have to agree: the tags tell the model what she is doing, the skeleton tells the ControlNet where her limbs go. All ten PNGs are in `skeletons/` beside this file.

| Pose | Tags to type | Skeleton file | What it reads as |
| --- | --- | --- | --- |
| Arms at sides | `arms_at_sides` | `arms_at_sides.png` | Arms hanging loose at her sides. Relaxed, open, or maybe just spaced out. |
| Folded hands | `own_hands_together` | `folded_hands.png` | Hands clasped together in front of her. Demure, careful, or maybe just shy. |
| Hand on hip | `hand_on_own_hip, arm_at_side` | `hand_on_hip.png` | One hand cocked on her hip, the other arm loose. Self-assured, sassy, or maybe just aloof. |
| Hands on hips | `hands_on_own_hips` | `hands_on_hips.png` | Both hands planted on her hips, elbows out. Bold, commanding, or maybe just silly. |
| Crossed arms | `crossed_arms` | `crossed_arms.png` | Arms folded across her chest. Closed off, defensive, or maybe just confident. |
| Clutching arm | `hand_on_own_arm, arm_at_side` | `clutching_arm.png` | One hand gripping her own opposite arm. Nervous, self-soothing, or maybe just creepy. |
| One arm crossed, other up | `arm_under_breasts, hand_up` | `one_arm_crossed_other_up.png` | One arm braced under her chest, the other hand raised near her face. Theatrical, thoughtful, or maybe just a worrier. |
| Hand on chest | `hand_on_own_chest` | `hand_on_chest.png` | One hand resting over her heart. Earnest, easygoing, or maybe just arrogant. |
| Bold | `legs_apart, arms_at_sides` | `bold.png` | Standing with legs apart and arms at her sides. Assertive, poised, or maybe just rude. |
| Hands behind back | `arms_behind_back` | `hands_behind_back.png` | Hip cocked and hands hidden behind her back. Playful, coy, or maybe a little mysterious. |

## Expressions

Seven emotions, in the order the game uses them: `neutral`, `happy`, `sad`, `angry`, `surprised`, `embarrassed`, `aroused`. Each has its own box, and each box is appended to the shared prompt for that one face pass.

The game builds each list by taking at most one tag from each of the first four groups and zero to two from "other":

| Group | Pick | Values |
| --- | --- | --- |
| General | 0 or 1 | `expressionless`, `happy`, `(smug:0.5)`, `(angry:0.5)`, `(sad:0.7)`, `(embarrassed:0.5)`, `(surprised:0.7)`, `naughty_face` |
| Eyebrows | 0 or 1 | `raised_eyebrow`, `raised_eyebrows`, `furrowed_brow` |
| Mouth | 0 or 1 | `clenched_teeth`, `(frown:0.5)`, `parted_lips`, `light_smile`, `smirk`, `grin`, `open_mouth` |
| Eyes | 0 or 1 | `(half-closed_eyes:0.7)`, `(wide-eyed:0.6)`, `one_eye_closed`, `looking_at_viewer`, `sideways_glance` |
| Other | 0 to 2 | `tears`, `blush` |

Skip any group that has nothing fitting. Not every emotion needs a general tag, and the seven should read as genuinely different from each other.

The weights in those tags are **lower bounds**, not fixed values. For a more expressive character, raise one as far as 0.9 and write your own number in: `(sad:0.7)` becomes `(sad:0.9)`.

One hard rule: **the `neutral` box must contain `looking_at_viewer` and must never contain `sideways_glance`.** The game cuts her portrait out of the neutral sprite, and a portrait looking off to one side is no good. The game rewrites its own characters' neutral tags to guarantee this; here it is on you.

## Seed

The **Character seed** box holds one number, and that number drives every image in the run: the base frames, the seven sprites of every outfit, and the CGs. That is what makes the same girl turn up in all of them.

Keep the control set to `fixed`. If it rerolls between images you get a different person in every folder.

Change the seed when you want a different girl out of the same description. Everything else can stay exactly as it is.

## NSFW content

Everything below this heading is adult content. The workflow ships with all of it switched off, and on the canvas both parts sit under cover notes and are collapsed. To see either one, drag its cover note aside, then click the dot at the left of a collapsed node's title bar to open it. The CG group has 34 collapsed nodes, so there drag a selection box around the group and press Alt+C to expand them all at once. Alt+C on the same selection collapses them again.

### The nude outfit

The game treats nudity as a fourth outfit with a fixed tag list. On the canvas the same line sits on the collapsed note under the first cover note. Paste it into the **Outfit 4** box and unmute the box with Ctrl+M:

```
nude: nude, nipples, navel, pussy, bare_arms, bare_legs, bare_shoulders, barefoot
```

You get a `nude/` folder with the same eight files as any other outfit.

### The CG line

The CG group renders eight adult illustrations at 1216x832 instead of the portrait sprite format, with no background removal and no ControlNet on the sampling pass. It is disabled by default in a way that keeps it out of validation. Once you have expanded the group as described above, switching it on takes three steps:

1. Drag one wire from the **Appearance** box to the **CG appearance in** node, which ships with that input unconnected.
2. Select the **Save CG** node.
3. Press Ctrl+M to unmute it.

Then queue as usual. The eight positions fan out the same way the outfits do, one press for all of them, into `output/VenusUniversity/cg/`.

The eight positions and the tags that describe them:

| Position | Tags |
| --- | --- |
| `nude_foreplay` | `nude, solo, nipples, pussy, on_back` |
| `nude_foreplay_after` | `nude, solo, nipples, pussy, on_back, after_sex` |
| `sex` | `nude, solo_focus, nipples, penis, pussy, on_back, vaginal` |
| `sex_after` | `nude, solo_focus, nipples, pussy, on_back, vaginal, after_sex, cum_on_body` |
| `handjob` | `nude, pov, handjob, penis` |
| `handjob_after` | `nude, pov, handjob, penis, after_sex, facial` |
| `fellatio` | `nude, pov, fellatio, penis` |
| `fellatio_after` | `nude, pov, handjob, after_fellatio, facial, cum_in_mouth, open_mouth, penis` |

Each name ending in `_after` is the aftermath of the one before it, which is why the two of each pair share most of their tags.

**How the game pairs expressions with these, and what this workflow does instead.** In the game each CG carries a face from the character's own expression list: her `aroused` tags for the four "during" positions, her `happy` tags for the four `_after` ones. The two fellatio positions drop any mouth tag from that list, since her mouth is busy. The face pass for an `_after` position also carries whichever of `after_sex`, `after_fellatio` and `facial` appear in its position tags, and `fellatio_after` adds `open_mouth` on top.

This workflow has a single **CG expression** box, prefilled with the aroused tags, applied to all eight. If you want the game's pairing, run the four "during" positions with an aroused box, then mute those and rerun the four `_after` positions with a happy box. For the two fellatio positions, delete any mouth tag from the box.

## How this differs from the game

The app runs one graph per image. This workflow does everything in one press, which means a handful of deliberate shortcuts:

1. **Background removal runs once per outfit, not once per sprite.** The mask is cut from that outfit's base frame and reused for all seven expressions. The face region each expression pass repaints is eroded slightly and added back into the keep mask, so a repainted face never gets cut off, but a sprite whose face pass moved a lot of hair can show a small difference at the edge.
2. **Each face pass prompts with its own outfit.** In the game every expression pass describes the everyday outfit no matter which wardrobe it is painting. Here the outfit box that produced the frame is the one in the prompt, which is more consistent but not identical output.
3. **One CG expression box** instead of the per-position pairing described above.
4. **The swimsuit negatives are not applied automatically.** One shared negative box serves every outfit; see the swimsuit tip above.
5. **The background threshold is aggressive on purpose.** Rembg runs at `0.9`, which cuts a halo off the figure rather than leaving one, because the game paints its own background behind the sprite and repairs the edge. If you want the sprites for something else and are losing thin details like loose hair or a fingertip, lower the threshold on the **Background cut (rembg)** node to `0.7` or `0.8` and accept the odd stray pixel or grey fringe.

### Speed tips

- **Bypass the four hires nodes** with Ctrl+B: the ESRGAN upscale, the 0.35 rescale, the VAE encode and the refining sampler. That roughly halves the base pass, at some cost in sharpness and fine detail.
- **Bypass the hand detailer** if the hands come out fine for your pose. It is a full extra sampling pass per outfit.
- **Mute expression lanes** you do not need by muting that lane's save node (**Save: happy** and so on), and **mute outfit boxes** you are not rendering. Muting boxes is also the fix for running out of VRAM: render one outfit at a time.
- The CG group is already off, so it costs nothing until you wire it up.

## Using the images in the game

The game reads a character's images off her own folder, with a fixed layout:

```
expressions/neutral.base.png        the everyday base frame
expressions/<emotion>.png           the seven everyday sprites
expressions/profile.png             her portrait
outfits/pe/neutral.base.png         and the same for swim and nude
outfits/pe/<emotion>.png
cg/<position>.png
```

So a run maps onto it directly: the folder from your first outfit box is `expressions/`, and the other outfit folders are `outfits/pe/`, `outfits/swim/` and `outfits/nude/`. `profile.png` is not something you render; the app cuts it out of the neutral sprite itself.

The supported way to make a character is the game's own **Manage Characters** screen, which drives all of this for you, seeds included. This workflow is for people who would rather work in ComfyUI directly and handle their own files.

## Troubleshooting

**A node is red after loading the JSON.** Its custom node pack is not installed. Install the three packs listed under Requirements, restart ComfyUI and reload the page.

**A node loads but a model dropdown is red or empty.** The file is missing, in the wrong folder, or has the wrong name. Check it against the model table, including the literal `%20` characters in the face segmentation filename.

**The Pose skeleton node says the file is missing.** The skeleton PNGs live next to this file in `skeletons/` and are not inside the workflow. Press the node's upload button and pick the one for your pose.

**Out of memory.** Mute all but one outfit box and run them one at a time. If it still will not fit, bypass the four hires nodes and the hand detailer as described under Speed tips.

**Files landed in a folder with no name.** An outfit line was missing its `name:` prefix, so there was no folder name to use. Every outfit box needs the name and the colon before its tags.

**Everything came out as a different girl in each folder.** The Character seed control rerolled. Set it back to `fixed`.
