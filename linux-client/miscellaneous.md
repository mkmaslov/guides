# Miscellaneous instructions

### Synchronize two folders

`rsync --archive --delete --progress <SOURCE-PATH> <TARGET-PATH>`

Comment:
* `--archive` - recursive file tree, preserves symlinks/permissions/owner/times/devices/special files
* `--delete` - removes those files from the target, that were previously removed from the source
* `--progress` - display sync status (the time estimate is wrong!)

---

### Overlay two `.pdf` files

`qpdf --overlay <OVERPLAY-FILE> --repeat=1 -- <SOURCE-FILE> <TARGET-FILE>`

---

### Add annotations to a PDF file

Create a watermark tile. On MacOS:
```
magick -background none -font "/System/Library/Fonts/Supplemental/Arial.ttf" -fill "rgba(0,0,0,0.15)" -pointsize 48 label:"SOME TEXT" -rotate 45 -trim +repage -bordercolor none -border 80x80 watermark_tile.png
```
Add `watermark_tile.png` as overlay:
```
magick -density 200 input.pdf -alpha set -tile "watermark_tile.png" -draw "rectangle 0,0 10000,10000" output.pdf
```

---