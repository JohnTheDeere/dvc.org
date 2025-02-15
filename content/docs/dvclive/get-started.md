# Get Started

DVCLive is a Python library for logging machine learning parameters, metrics and
other metadata in simple file formats, which is fully compatible with DVC, the
[VS Code Extension](https://marketplace.visualstudio.com/items?itemName=Iterative.dvc),
and [Iterative Studio](https://studio.iterative.ai/).

## Set up DVCLive

First of all, you need to add DVCLive to your Python script:

<toggle>
<tab title="Keras">

```python
# train.py
from dvclive.keras import DvcLiveCallback
...
model.fit(
  train_dataset, validation_data=validation_dataset,
  callbacks=[DvcLiveCallback()])
```

</tab>

<tab title="Hugging Face">

```python
# train.py
from dvclive.huggingface import DvcLiveCallback

. . .

 trainer = Trainer(
    model, args,
    train_dataset=train_data,
    eval_dataset=eval_data,
    tokenizer=tokenizer,
    compute_metrics=compute_metrics,
)
trainer.add_callback(DvcLiveCallback())
trainer.train()
```

</tab>
<tab title="Pytorch Lightning">

```python
# train.py
from dvclive.lightning import DvcLiveLogger

. . .
dvclive_logger = DvcLiveLogger()

trainer = Trainer(logger=dvclive_logger)
trainer.fit(model)
```

</tab>

<tab title="Python API">

```python
# train.py
from dvclive import Live

live = Live()

live.log_param("epochs", NUM_EPOCHS)

for epoch in range(NUM_EPOCHS):
    train_model(...)
    metrics = evaluate_model(...)

    for metric_name, value in metrics.items():
        live.log(metric_name, value)

    live.next_step()
```

</tab>
</toggle>

Check the [ML Frameworks](/doc/dvclive/api-reference/ml-frameworks) page for
more details and other supported frameworks.

### Run the script

Once you have added DVCLive to our python code, you can run the script as you
would usually do:

```dvc
$ python train.py
```

DVCLive will generate a [report](/doc/dvclive/outputs#report) in
`dvclive/report.html` containing all the logged data. It will be automatically
updated during training on each `step` update:

![HTML report](/img/dvclive-html.gif).

In addition, the logged data will be stored in plain text files. For this simple
example, it would look as follows:

```
dvclive.json
dvclive
├── scalars
│       └── metric.tsv
└── report.html
```

<admon type="info" icon="book">

You can find more details about the structure of files generated by DVCLive in
the [Output Folder Structure](/doc/dvclive/outputs) page.

</admon>

## DVC Integration

To build on the previous section, you can optionally wrap our code with DVC to
run [DVC experiments](/doc/user-guide/experiment-management/) and enable new
ways to compare and visualize our experiments.

You can find below an example `dvc.yaml` containing a single stage `train`, that
runs the script from previous steps and tracks the
[DVCLive outputs](/doc/dvclive/outputs) as `dvc metrics` and `dvc plots`:

```yaml
stages:
  train:
    cmd: python train.py
    deps:
      - train.py
    metrics:
      - dvclive.json:
          cache: false
    plots:
      - dvclive/scalars:
          cache: false
```

### Run DVC Experiments

You can now
[run experiments](/doc/user-guide/experiment-management/running-experiments)
with `dvc exp run`:

```dvc
$ dvc exp run
Running stage 'train':
> python train.py
...
```

### Compare and Visualize DVC Experiments

After following the above steps, you have enabled different ways of monitoring
the training, comparing, and visualizing the experiments:

#### DVC CLI

You can use `dvc exp show` and `dvc plots` to compare and visualize metrics,
parameters and plots across experiments.

<admon type="info" icon="book">

Learn more in the
[Comparing Experiments](/doc/user-guide/experiment-management/comparing-experiments)
and [Visualizing Plots](/doc/user-guide/experiment-management/visualizing-plots)
pages of the user guide.

</admon>

#### DVC Extension for Visual Studio Code

If you have installed the
[DVC Extension for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=Iterative.dvc),
you can monitor the training and compare and visualize experiments using the
[`Experiments`](https://github.com/iterative/vscode-dvc/blob/main/extension/resources/walkthrough/experiments-table.md)
and
[`Plots`](https://github.com/iterative/vscode-dvc/blob/main/extension/resources/walkthrough/plots.md)
views.

![Experiments view](https://github.com/iterative/vscode-dvc/raw/main/extension/resources/walkthrough/images/experiments-table.png)
![Plots view](https://github.com/iterative/vscode-dvc/raw/main/extension/resources/walkthrough/images/plots-trends.png)

### Share Results

After the experiment has finished and you have committed and pushed the results,
[Iterative Studio](/doc/studio) will automatically parse the outputs generated
by DVCLive, allowing you to
[share your experiments online](https://dvc.org/doc/studio/get-started):

![Studio view](/img/dvclive-studio-plots.png)
