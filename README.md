# DICOM to video
[![linters](https://github.com/andreoss/dicom/actions/workflows/linters.yml/badge.svg)](https://github.com/andreoss/dicom/actions/workflows/linters.yml)

Convert [DICOM](https://en.wikipedia.org/wiki/DICOM) files such as magnetic resonance and computer tomography images, fluoroscopy and ultrasound images to a video file.

## Requirements
* ImageMagick
* ffmpeg

## Usage

You need to specify an input directory, which is walked recursively.
If `--weigth` and `--height` not specified, maximal available values for both will be used.
Frame rate can be specified by `--rate`.

## Example

```
$ ./dicom --input ~/DICOM/ --overwrite --output ~/result.mp4 --rate 6 --width 600 --height 400
```
