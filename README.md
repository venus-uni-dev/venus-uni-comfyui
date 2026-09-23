# Venus University character workflow

`VenusUniversityCharacter.workflow.json` is a ComfyUI workflow based off the workflows I use to generate characters in my game Venus University: 

https://venus-dev.itch.io/venus-university

The full game uses a similar workflow populated programatically to spawn a full set of character images from a single user prompt. The character creator comes with additional features like hand inpainting and single image regen, but I got requests to upload a standalone workflow, so here it is.

Please note that this uses an IllustriousXL model and LORA by default. If you change the model, you may need to change the controlnet model as well, since it is specifically for illustrious.

## Sprite Generation

<img width="1280" height="720" alt="Untitled-1" src="https://github.com/user-attachments/assets/c2db3ee6-b814-4144-853a-6be159f0a234" />

This workflow generates 7 expression images for up to 4 outfits with a transparent background + full body pose. There's also an off-by-default group for generating NSFW CG of the same character.

## Required Nodes

- [ComfyUI-Impact-Pack](https://github.com/ltdrdata/ComfyUI-Impact-Pack) (Face Detailer, Make List)
- [ComfyUI-Impact-Subpack](https://github.com/ltdrdata/ComfyUI-Impact-Subpack) (the Ultralytics detector provider)
- [ComfyUI-Inspyrenet-Rembg](https://github.com/john-mnz/ComfyUI-Inspyrenet-Rembg) (background removal)

I recommend using these model files. The nodes are set to the names that I renamed them to in Venus University, but you can always just put the models in the appropriate folders and select the correct model manually instead of renaming.

| Download page | Downloaded name | Rename to | Put in |
| --- | --- | --- | --- |
| [Nova Anime XL](https://civitai.com/models/376130/nova-anime-xl), version IL v19.0 | `NovaAnimeILV190.safetensors` | `novaAnimeXL_ilV190.safetensors` | `models/checkpoints` |
| [USNR STYLE](https://civitai.com/models/176554/usnr-style?modelVersionId=1552087), version USNR_STYLE_ILL_V1.0 | `USNR_STYLE_ILL_v1_lokr3-000024.safetensors` | `usnrStyle.safetensors` | `models/loras` |
| [Illustrious-XL ControlNet Openpose](https://civitai.com/models/1359846/illustrious-xl-controlnet-openpose), version v1.0 | `illustriousXL_v10.safetensors` | `illustriousOpenpose.safetensors` | `models/controlnet` |
| [Real-ESRGAN v0.2.2.4](https://github.com/xinntao/Real-ESRGAN/releases/tag/v0.2.2.4) | `RealESRGAN_x4plus_anime_6B.pth` | keep the name | `models/upscale_models` |
| [Bingsu/adetailer](https://huggingface.co/Bingsu/adetailer) | `hand_yolov9c.pt` | keep the name | `models/ultralytics/bbox` |
| [Anzhc/Anzhcs_YOLOs](https://huggingface.co/Anzhc/Anzhcs_YOLOs) | `Anzhc Face seg 640 v2 y8n.pt` | `Anzhc%20Face%20seg%20640%20v2%20y8n.pt` | `models/ultralytics/segm` |

## Setup

1. Install the three node packs and restart ComfyUI.
2. Put the six model files into the appropriate folders
3. Drop `VenusUniversityCharacter.workflow.json` onto the ComfyUI canvas.
4. If a node loads but one of its model dropdowns is red or empty, the file is in the wrong folder or has the wrong name, check the table
5. On the **Pose skeleton** node, upload one of the provided pose images and make sure the pose tags match

Detailed instructions are on the START HERE section in the workflow, please read everything carefully before generation

<img width="1994" height="581" alt="image" src="https://github.com/user-attachments/assets/9c6115f5-3d0c-4d77-9851-1f45746886cd" />
