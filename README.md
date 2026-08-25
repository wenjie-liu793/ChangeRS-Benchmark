# Changers Benchmark

Changers Benchmark is a 3,000-question multiple-choice benchmark for
evaluating RGB-visible remote-sensing change understanding in vision-language
models.

## Download

Download the self-contained dataset from Google Drive:

- **Google Drive:** [Changers Benchmark](https://drive.google.com/file/d/1hsxs2iRJmddQzum0OCZn6KXlbJrPmC1r/view?usp=sharing)
- **File:** `changers-benchmark.zip`
- **Package size:** approximately 6.8 GB
- **Archive format:** ZIP64


## Dataset contents

```text
changers-benchmark/
├── changers.jsonl
├── stats.json
├── images/          # 5,800 available image files
├── masks/           # 3,000 auxiliary change masks
├── semantic/        # 2,508 auxiliary semantic-label images
├── README.md
└── SHA256SUMS
```

The JSONL contains 6,000 T1/T2 path fields. Of these, 200 paths are
deliberately absent across 100 E-correct missing-input controls. The package
must not add placeholder images for these records.

## Download and extract

Download the archive from Google Drive:

```text
changers-benchmark.zip
```

Extract it on Linux or macOS:

```bash
unzip changers-benchmark.zip
cd changers-benchmark
```

Because the archive is larger than 4 GB, it uses ZIP64. A recent version of
7-Zip is recommended on Windows and can also be used on Linux or macOS:

```bash
7z x changers-benchmark.zip
```

## Verify integrity

The package includes checksums for every dataset file:

```bash
cd changers-benchmark
sha256sum --check SHA256SUMS
```

## JSONL format

Each line is one JSON object. Important fields include:

- `id`: unique question ID in the form `changers_0001` through
  `changers_3000`.
- `image_t1`, `image_t2`: paths relative to the extracted package directory.
- `question`: multiple-choice question.
- `options`: candidate answers A-F.
- `answer`: ground-truth option letter.
- `capability`: evaluated change-understanding capability.
- `source_dataset`: source remote-sensing dataset.
- `subset`: `visual_change` or `input_validity`.
- `auxiliary_paths.change_mask`: auxiliary change-mask path.
- `auxiliary_paths.semantic_t1`, `semantic_t2`: auxiliary semantic-label paths.

Example paths:

```text
images/example_t1.png
images/example_t2.png
masks/example_mask.png
semantic/example_sem_t1.png
```

## Python usage

```python
import json
from pathlib import Path

from PIL import Image

root = Path("changers-benchmark")
jsonl_path = root / "changers.jsonl"

with jsonl_path.open("r", encoding="utf-8") as handle:
    for line in handle:
        item = json.loads(line)

        t1_path = root / item["image_t1"]
        t2_path = root / item["image_t2"]

        # Missing image files are part of the input-validity evaluation.
        # Do not manufacture placeholder images.
        image_t1 = Image.open(t1_path).convert("RGB") if t1_path.exists() else None
        image_t2 = Image.open(t2_path).convert("RGB") if t2_path.exists() else None

        item_id = item["id"]
        question = item["question"]
        options = item["options"]

        # Send the available image media, question, and options to the model.
        # Store one predicted option letter (A-F) for item_id.
```

Expected prediction format:

```json
{
  "changers_0001": "D",
  "changers_0002": "A"
}
```

## Evaluation rules

- Normal `visual_change` questions contain two available images.
- The 100 E-correct missing-input controls deliberately have no available T1
  or T2 image files. Send the question and options without image media.
- F-correct mismatched-area controls contain two available images.
- `masks/` and `semantic/` are construction and review evidence. Do not send
  them to the model during normal visual-pair evaluation.
- Report normal-question accuracy and input-validity E/F accuracy separately.

The JSONL includes ground-truth answers and therefore should not be presented
as a hidden test set without first creating a question-only release.

## Statistics

- Questions: 3,000
- Available T1/T2 image files: 5,800
- Intentionally absent image references: 200
- Missing-input control records: 100
- Change masks: 3,000
- Semantic-label images: 2,508

See `stats.json` for capability and source-dataset distributions.

## Licensing

The benchmark contains derived samples from multiple public remote-sensing
datasets. Users and redistributors must follow the license and attribution
requirements of the corresponding source datasets. This release does not
replace upstream dataset licenses.
