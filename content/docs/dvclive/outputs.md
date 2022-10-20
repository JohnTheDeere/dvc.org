# Output Folder Structure

DVCLive will store the logged data under the folder (`path`) passed to
[`Live()`](/doc/dvclive/api-reference/live). If not provided, `dvclive` will be
used by default.

The resulting structure of the folder depends on the methods used for logging
data:

| Method                                                            | Writes to                                        |
| ----------------------------------------------------------------- | ------------------------------------------------ |
| [live.log](/doc/dvclive/api-reference/live/log)                   | `dvclive/plots/metrics` & `dvclive/metrics.json` |
| [live.log_image](/doc/dvclive/api-reference/live/log_image)       | `dvclive/plots/images`                           |
| [live.log_param](/doc/dvclive/api-reference/live/log_param)       | `dvclive/params.yaml`                            |
| [live.log_sklearn_plot](/doc/dvclive/api-reference/live/log_plot) | `dvclive/plots/sklearn`                          |
| [live.make_report](/doc/dvclive/api-reference/live/make_report)   | `dvclive/report.{md/html}`                       |
| [live.make_summary](/doc/dvclive/api-reference/live/make_summary) | `dvclive/metrics.json`                           |

## Example

To illustrate with an example, given the following script:

```python
import random

from dvclive import Live
from PIL import Image

EPOCHS = 2

live = Live()

live.log_param("epochs", EPOCHS)

for i in range(EPOCHS):
    live.log("metric", i + random.random())
    live.log("nested/metric", i + random.random())

    img = Image.new("RGB", (50, 50), (i, i, i))
    live.log_image("img.png", img)

    live.next_step()

live.log_sklearn_plot("confusion_matrix", [0, 0, 1, 1], [0, 1, 0, 1])

live.summary["additional_metric"] = 1.0
live.make_summary()
live.make_report()
```

The resulting structure will be:

```
dvclive
├── metrics.json
├── params.yaml
├── plots
│   ├── images
│   │   └── img.png
│   ├── metrics
│   │   ├── metric.tsv
│   │   └── nested
│   │       └── metric.tsv
│   └── sklearn
│       └── confusion_matrix.json
└── report.html
```
