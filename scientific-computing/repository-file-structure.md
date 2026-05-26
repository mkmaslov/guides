# `git` repository file structure

*Repository/folder/file names (use dashes):* `some-repository/some-folder/some-subfolder/some-file.extension`

These are the possible main folders inside the repository's root directory:

- `ARCHIVE`: storage for redundant files.<br>
This folder should be erased upon project completion, use it to store files that are not needed directly, but can be handy when working on a project / debugging something.
- `cluster`: everything related to HPC calculations, **except the data**.
- `data`: storage for the temporary / final data.
- `figures`: rendered figures for the manuscript.<br>
*Use subfolders*: `poster` or `talk` to store the respective images.
- `manuscript`: either LaTeX files or an Overleaf backup (zip archive).
- `notes`: discussion notes, handwritten notes, photos.<br>
*Use subfolders:* `YYMMDD-short-description/`
- `poster`: Inkscape/Illustrator/LaTeX files for the poster.
- `references`: papers/books that are relevant for the project (even if uncited).<br>
*Naming scheme*: `YYYY-FirstAuthor-(theory|experiment)-description.pdf`
- `resources`: external libraries or scripts.
- `scripts`: do **not** store the entire codebase in single notebook / script file.
- `talk`: presentation(s) about the project. Images should be stored separately (in `figures` folder).

What should `.gitignore` include:
- `.gitignore`
- `*.DS_Store*`: to avoid syncing MacOS temporary folders
- `IGNORE-*`: to manually exclude unwanted files.
- `.ipynb_checkpoints/`: to avoid syncing temporary files from Jupyter.
- `__pycache__/`: to avoid syncing Python cache.
- `.vscode/*`: to avoid syncing local VS Code settings.