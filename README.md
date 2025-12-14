# ML-tree — Architecture

This repository contains a PlantUML diagram (file `di`) describing the project's architecture. This README shows the diagram on the repository home page in one of two ways:

- Option A (recommended): Commit a rendered SVG (`diagram.svg`) next to this README and embed it.
- Option B: Use the PlantUML public server to render the diagram on the fly (no image commit required).
- Option C: Automatically generate `diagram.svg` on every push using GitHub Actions.

---

## Option A — Commit a rendered SVG (recommended)

1. Copy `di` to `diagram.puml` (optional but convenient).
2. Generate an SVG locally (requires Java + PlantUML + Graphviz):

```bash
# if you have plantuml CLI:
plantuml -tsvg diagram.puml

# or using the plantuml jar:
java -jar plantuml.jar -tsvg diagram.puml
```

3. Commit the generated file:

```bash
git add diagram.svg README.md
git commit -m "Add architecture diagram"
git push
```

4. The image will be displayed in this README:

![Architecture](diagram.svg)

---

## Option B — Use PlantUML server (no commit)

1. Produce the PlantUML encoded string from the `di` file (the encoding is deflate+custom base64). You can use the helper script `encode_plantuml.py` (example provided below) or any PlantUML encoding tool.
2. Replace `<ENCODED>` in the URL below with the encoded string:

```markdown
![Architecture](https://www.plantuml.com/plantuml/svg/<ENCODED>)
```

Note: Relying on the public PlantUML server means the image is loaded from an external service. For private repos or if you prefer no external dependency, use Option A or C.

---

## Option C — Auto-generate on push with GitHub Actions

Create `.github/workflows/generate-plantuml.yml` with this job. It will generate `diagram.svg` from `di` on each push and commit it back:

```yaml
name: Generate PlantUML diagram

on:
  push:
    paths:
      - "di"
      - ".github/workflows/generate-plantuml.yml"

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          persist-credentials: true

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          java-version: '17'

      - name: Download PlantUML jar
        run: |
          curl -sSL -o plantuml.jar https://plantuml.com/plantuml.jar

      - name: Generate SVG from PlantUML
        run: |
          if [ ! -f diagram.puml ] && [ -f di ]; then
            cp di diagram.puml
          fi
          java -jar plantuml.jar -tsvg diagram.puml

      - name: Commit generated SVG if changed
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "41898282+github-actions[bot]@users.noreply.github.com"
          git add -A
          if ! git diff --cached --quiet; then
            git commit -m "chore: regenerate PlantUML diagram.svg"
            git push
          else
            echo "No changes to commit"
          fi
```

---

## Helper: Python encoder for PlantUML server (optional)

Save the following as `encode_plantuml.py` and run:

```bash
python3 encode_plantuml.py di
# prints the encoded string — paste after /plantuml/svg/
```

(If you want, I can add this file to the repo for you.)

---

## Notes and recommendations

- Committing the SVG (Option A) provides the most reliable and fastest rendering for README viewers.
- Using the PlantUML server (Option B) avoids committing generated artifacts, but it depends on an external service and requires generating the encoded URL.
- The GitHub Actions approach (Option C) gets the best of both worlds — source (`di`) is authoritative, and `diagram.svg` is updated automatically and displayed in README.

---

If you want, I can:
- generate `diagram.svg` locally from your `di` content and provide the file ready to commit, or
- create the GitHub Actions workflow and a starter `README.md` commit that references the generated file and open a branch/PR — tell me which and I will prepare the files. 
