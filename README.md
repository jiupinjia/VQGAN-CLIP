# VQGAN-CLIP Overview
A repo for running VQGAN+CLIP locally. This started out as a Katherine Crowson VQGAN+CLIP derived Google colab notebook.

Original notebook: [![Open In Colab][colab-badge]][colab-notebook]

[colab-notebook]: <https://colab.research.google.com/drive/1ZAus_gn2RhTZWzOWUpPERNC0Q8OhZRTZ>
[colab-badge]: <https://colab.research.google.com/assets/colab-badge.svg>

Some example images:

<img src="./samples/Cartoon3.png" width="256px"></img><img src="./samples/Cartoon.png" width="256px"></img><img src="./samples/Cartoon2.png" width="256px"></img>
<img src="./samples/Bedroom.png" width="256px"></img><img src="./samples/DemonBiscuits.png" width="256px"></img><img src="./samples/Football.png" width="256px"></img>
<img src="./samples/Fractal_Landscape3.png" width="256px"></img><img src="./samples/Games_5.png" width="256px"></img>

Environment:
* Tested on Ubuntu 20.04
* GPU: Nvidia RTX 3090
* Typical VRAM requirements:
  * 24 GB for a 900x900 image
  * 10 GB for a 512x512 image
  *  8 GB for a 380x380 image

Still a work in progress - I've not actually tested everything yet :)

# Set up
Example set up using Anaconda to create a virtual Python environment with the prerequisites:

```
conda create --name vqgan python=3.9
conda activate vqgan

pip install torch==1.9.0+cu111 torchvision==0.10.0+cu111 torchaudio==0.9.0 -f https://download.pytorch.org/whl/torch_stable.html
pip install ftfy regex tqdm omegaconf pytorch-lightning IPython kornia imageio imageio-ffmpeg einops 

git clone https://github.com/openai/CLIP
git clone https://github.com/CompVis/taming-transformers.git
```
You will also need at least 1 VQGAN pretrained model. E.g.
```
mkdir checkpoints
curl -L -o checkpoints/vqgan_imagenet_f16_16384.yaml -C - 'http://mirror.io.community/blob/vqgan/vqgan_imagenet_f16_16384.yaml' #ImageNet 16384
curl -L -o checkpoints/vqgan_imagenet_f16_16384.ckpt -C - 'http://mirror.io.community/blob/vqgan/vqgan_imagenet_f16_16384.ckpt' #ImageNet 16384
```
By default, the model .yaml and .ckpt files are expected in the `checkpoints` directory.
See https://github.com/CompVis/taming-transformers for more information on datasets and models.

# Run
To generate images from text, specify your text prompt as shown in the example below:
```
python generate.py -p "A painting of an apple in a fruit bowl"
```
<img src="./samples/A_painting_of_an_apple_in_a_fruitbowl.png" width="256px"></img>

## Multiple prompts
Text and image prompts can be split using the pipe symbol in order to allow multiple prompts. For example:

```
python generate.py -p "A painting of an apple in a fruit bowl | psychedelic | surreal | weird"
```
<img src="./samples/Apple_weird.png" width="256px"></img>

Image prompts can be split in the same way. For example:

```
python generate.py -p "A picture of a bedroom with a portrait of Van Gogh" -ip "samples/VanGogh.jpg | samples/Bedroom.png"
```

## "Style Transfer"
An input image with style text and a low number of iterations can be used create a sort of "style transfer" effect. For example:

```
python generate.py -p "A painting in the style of Picasso" -ii samples/VanGogh.jpg -i 80 -se 10 -opt AdamW -lr 0.25
```

| Output        | Style         |
| ------------- | ------------- |
| <img src="./samples/vvg_picasso.png" width="256px"></img> | Picasso |
| <img src="./samples/vvg_sketch.png" width="256px"></img>  | Sketch  |
| <img src="./samples/vvg_psychedelic.png" width="256px"></img>  | Psychedelic  |

## Feedback example
By feeding back the generated images and making slight changes, some interesting effects can be created.

The example `zoom.sh` shows this by applying a zoom and rotate to generated images, before feeding them back in again.
To use `zoom.sh`, specifying a text prompt, output filename and number of frames. E.g.
```
./zoom.sh "A painting of a red telephone box spinning through a time vortex" Telephone.png 150
```
<img src="./samples/zoom.gif" width="256px"></img>

## Random text example
Use `random.sh` to make a batch of images from random text. Edit the text and number of generated images to your taste!
```
./random.sh
```


## Advanced options
To view the available options, use "-h".
```
python generate.py -h
```

```
usage: generate.py [-h] [-p PROMPTS] [-o OUTPUT] [-i MAX_ITERATIONS] [-ip IMAGE_PROMPTS]
[-nps [NOISE_PROMPT_SEEDS ...]] [-npw [NOISE_PROMPT_WEIGHTS ...]] [-s SIZE SIZE]
[-ii INIT_IMAGE] [-iw INIT_WEIGHT] [-m CLIP_MODEL] [-conf VQGAN_CONFIG]
[-ckpt VQGAN_CHECKPOINT] [-lr STEP_SIZE] [-cuts CUTN] [-cutp CUT_POW] [-se DISPLAY_FREQ]
[-sd SEED] [-opt OPTIMISER]
```

```
optional arguments:
  -h, --help            show this help message and exit
  -p PROMPTS, --prompts PROMPTS
                        Text prompts
  -o OUTPUT, --output OUTPUT
                        Number of iterations
  -i MAX_ITERATIONS, --iterations MAX_ITERATIONS
                        Number of iterations
  -ip IMAGE_PROMPTS, --image_prompts IMAGE_PROMPTS
                        Image prompts / target image
  -nps [NOISE_PROMPT_SEEDS ...], --noise_prompt_seeds [NOISE_PROMPT_SEEDS ...]
                        Noise prompt seeds
  -npw [NOISE_PROMPT_WEIGHTS ...], --noise_prompt_weights [NOISE_PROMPT_WEIGHTS ...]
                        Noise prompt weights
  -s SIZE SIZE, --size SIZE SIZE
                        Image size (width height)
  -ii INIT_IMAGE, --init_image INIT_IMAGE
                        Initial image
  -iw INIT_WEIGHT, --init_weight INIT_WEIGHT
                        Initial image weight
  -m CLIP_MODEL, --clip_model CLIP_MODEL
                        CLIP model
  -conf VQGAN_CONFIG, --vqgan_config VQGAN_CONFIG
                        VQGAN config
  -ckpt VQGAN_CHECKPOINT, --vqgan_checkpoint VQGAN_CHECKPOINT
                        VQGAN checkpoint
  -lr STEP_SIZE, --learning_rate STEP_SIZE
                        Learning rate
  -cuts CUTN, --num_cuts CUTN
                        Number of cuts
  -cutp CUT_POW, --cut_power CUT_POW
                        Cut power
  -se DISPLAY_FREQ, --save_every DISPLAY_FREQ
                        Save image iterations
  -sd SEED, --seed SEED
                        Seed
  -opt OPTIMISER, --optimiser OPTIMISER
                        Optimiser (Adam, AdamW, Adagrad, Adamax)
```

# Citations

```bibtex
@misc{unpublished2021clip,
    title  = {CLIP: Connecting Text and Images},
    author = {Alec Radford, Ilya Sutskever, Jong Wook Kim, Gretchen Krueger, Sandhini Agarwal},
    year   = {2021}
}
```
```bibtex
@misc{esser2020taming,
      title={Taming Transformers for High-Resolution Image Synthesis}, 
      author={Patrick Esser and Robin Rombach and Björn Ommer},
      year={2020},
      eprint={2012.09841},
      archivePrefix={arXiv},
      primaryClass={cs.CV}
}
```
Katherine Crowson - https://github.com/crowsonkb

Public Domain images from Open Access Images at the Art Institute of Chicago - https://www.artic.edu/open-access/open-access-images
