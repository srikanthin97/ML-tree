# ML-tree — Architecture

This repository contains a PlantUML diagram file named `di` describing the project's architecture. This README (and the generated README.pdf) explains how the diagram is produced and how to get a rendered image.

What this commit provides
- README.md (this file)
- A GitHub Actions workflow that generates README.pdf from README.md and commits it to the repository on push.

Rendered diagram options

Option A — Commit a rendered SVG (recommended)
1. Generate an SVG locally from the PlantUML source `di` (or `diagram.puml`):

```
plantuml -tsvg di
```

2. Commit the produced `diagram.svg` next to this README. Use this to embed the image in the README and the PDF will include it when the workflow runs.

Option B — Use the PlantUML public server (no commit)

1. Encode the PlantUML source (deflate + custom base64) and paste the encoded string into this URL:

```
https://www.plantuml.com/plantuml/svg/<ENCODED>
```

2. Embed in README like:

```
![Architecture](https://www.plantuml.com/plantuml/svg/<ENCODED>)
```

Option C — Auto-generate diagram files and README.pdf via GitHub Actions
The included workflow generates README.pdf from README.md using the pandoc/latex Docker image and commits README.pdf back to the repository whenever README.md or the workflow file changes.

PlantUML source (file `di` in this repo)

@startuml
' Styling
skinparam classAttributeIconSize 0
skinparam packageStyle rectangle
skinparam shadowing false

' --- Data layer ---
package "data" {
  class DataLoader <<module>> {
    +load_data(path : string) : DataFrame
    +split(train_ratio : float) : (train, val, test)
  }
  class Dataset <<model>> {
    +__len__() : int
    +__getitem__(idx : int) : Sample
    -data : list
  }
  DataLoader --> Dataset : produces
}

' --- Preprocessing / Feature engineering ---
package "preprocessing" {
  class Preprocessor <<service>> {
    +fit_transform(data)
    +transform(data)
  }
  class FeatureEngineer <<service>> {
    +create_features(data)
    +select_features(data)
  }
  Preprocessor --> FeatureEngineer : uses
  FeatureEngineer --> Dataset : reads
}

' --- Models ---
package "models" {
  abstract class Model <<abstract>> {
    +train(data)
    +predict(x)
    +save(path)
    +load(path)
  }
  class SklearnModel <<adapter>> {
    +fit(X, y)
    +predict(X)
  }
  class TorchModel <<module>> {
    +forward(x)
    +train_step(batch)
  }
  class EnsembleModel <<composite>> {
    +add_submodel(m : Model)
    +predict(x)
  }
  SklearnModel -|> Model
  TorchModel -|> Model
  EnsembleModel --> Model : aggregates
}

' --- Training / Orchestration ---
package "training" {
  class Trainer <<orchestrator>> {
    +train_loop()
    +validate()
    -_save_checkpoint()
    -_load_checkpoint()
  }
  class Optimizer <<helper>> {
    +step()
  }
  class Scheduler <<helper>> {
    +step()
  }
  Trainer --> Model : trains
  Trainer --> Optimizer
  Trainer --> Scheduler
  Trainer --> Dataset
}

' --- Evaluation / Metrics ---
package "evaluation" {
  class Evaluator <<service>> {
    +evaluate(model, data)
    +compute_metrics()
  }
  class Metrics <<value-object>> {
    +accuracy
    +precision
    +recall
    +f1
    +loss
  }
  Evaluator --> Model
  Evaluator --> Dataset
  Evaluator --> Metrics
}

' --- CLI / Entrypoints ---
package "cli" {
  class CLI <<entrypoint>> {
    +main(args)
  }
  CLI --> Trainer
  CLI --> Evaluator
  CLI --> Config
}

' --- Utilities / Config / Logging ---
package "utils" {
  class Logger <<infra>> {
    +info(msg)
    +debug(msg)
  }
  class Config <<infra>> {
    +load(path)
    +get(key)
  }
  Config --> Logger
  Trainer --> Logger
  CLI --> Logger
}

' --- Persistence / IO ---
package "storage" {
  class Checkpointer <<infra>> {
    +save(model, path)
    +load(path)
  }
  class ArtifactStore <<infra>> {
    +upload(file)
    +download(id)
  }
  Trainer --> Checkpointer
  Checkpointer --> ArtifactStore
}

' --- Relationships / Notes ---
Dataset ..> Preprocessor : "input for"
Model ..> FeatureEngineer : "expects features"
SklearnModel ..> Optimizer : "may not require"
TorchModel ..> Optimizer : "uses"
EnsembleModel ..> Model : "composed of"

note left of DataLoader
This PlantUML represents a whole-repo ML project structure.
Replace placeholders (class names, attributes, methods) with
the actual classes and signatures from your code for accuracy.
end note

@enduml
