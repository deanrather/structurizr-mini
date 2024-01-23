<img src='./docs/banner.webp' width='512' alt='Structurizr Mini' />

A static site for C4 diagrams from a [Structurizr](https://structurizr.com) workspace. 👉 [Demo](https://bensmithett.github.io/structurizr-mini/) 👈

## Features

- Real Structurizr diagrams — uses the [same diagram renderer](https://github.com/structurizr/ui) as the [official tools](https://docs.structurizr.com/products)
- [Themes](https://structurizr.com/help/themes) just work
- Zoom and pan with mousewheel or trackpad (like Google Maps, Lucidchart, Miro, etc)
- Simplified UI with [quick navigation](https://docs.structurizr.com/ui/quick-navigation) and [fuzzy search](https://github.com/farzher/fuzzysort)
- PNG export
- Just the diagrams — [docs](https://docs.structurizr.com/dsl/docs) and [ADRs](https://docs.structurizr.com/dsl/adrs) in the workspace are ignored
- Customisable nav — link to your own supplemental docs, ADRs, source code, etc

## Instructions

1. Download and unzip the latest [release](https://github.com/bensmithett/structurizr-mini/releases) somewhere that can serve static files over HTTP
2. Replace `workspace.json` and (optionally) `nav.json` with your own.
3. Go to `http://[YOUR SERVER]/index.html`

How you get a `workspace.json` depends on your workflow, and the extent to which you're using Structurizr [Cloud](https://docs.structurizr.com/cloud) or [On-Premises](https://docs.structurizr.com/onpremises).

## Structurizr Cloud/On-Premises workflow

Use Structurizr CLI's [`pull`](https://docs.structurizr.com/cli/pull) to export a `workspace.json` that includes diagram layout information.

## Structurizr Mini workflow

1. Author a `workspace.dsl` locally, using [Structurizr Lite](https://structurizr.com/help/lite) or the [web DSL Editor](https://structurizr.com/dsl) to preview the resulting diagrams.
2. Generate a `workspace.json` based on your `workspace.dsl`
3. Publish diagrams to a static Structurizr Mini site (e.g. as part of a CI build)

⚠️ Note: while the Structurizr CLI can [export](https://docs.structurizr.com/cli/export) your local `workspace.dsl` to JSON, **it will not include any diagram layout info.** The following steps explain how to get a `workspace.json` that includes diagram layout info.

### Where does your diagram layout info come from?

Diagram layout is defined two ways: **automatically** and **manually**.

#### Automatic layout (i.e. `autoLayout`)

To get the actual layout information for your `autoLayout` views, Structurizr Lite/Cloud/On-Premises run [Graphviz](https://graphviz.org) on the server when you view a diagram.

```mermaid
sequenceDiagram
  Browser->>Server: Get layout information
  Server->>Server: Apply Graphviz
  Server->>Browser: Return layout information
  Browser->>Browser: Render diagram
```

As a static site without a server runtime, Mini can't do that. Luckily there's an [easy way](https://github.com/structurizr/cli/issues/62#issuecomment-999623728) to generate automatic layout information at build time:

1. [Install Graphviz](https://graphviz.org/download/) so the `dot` command is available
2. Create a wrapper DSL file (e.g. `graphviz.dsl`) that extends your JSON workspace, and applies graphviz.
```
workspace extends workspace.json {
  !script groovy {
    new com.structurizr.graphviz.GraphvizAutomaticLayout().apply(workspace);
  }
}
```
3. Use the CLI to export *that* workspace to JSON
```bash
structurizr-cli export -workspace graphviz.dsl -format json
```

This JSON workspace will have all the layout information that Structurizr Mini needs to render your `autoLayout` diagrams.

#### Manual layout

If you use Lite to locally edit the layout of non-`autoLayout` diagrams, Lite [auto-saves](https://docs.structurizr.com/lite/usage#auto-save) those edits to a local `workspace.json`.

You may wish to commit this file to version control. It's a `workspace.json` complete with layout information that Mini can use directly.

⚠️ Remember that layout info for a diagram only updates in Lite's `workspace.json` when you *actually view the individual diagram* in the browser. Things can get messy if you have many diagrams, or multiple people contributing changes, each updating their own local `workspace.json` according to which diagrams they view or edit.

If your workspace has a mix of automatic and manual layouts, you can still use the method above to generate automatic layout information for all diagrams based on the latest `workspace.dsl`, regardless of which individual diagrams contributors have viewed recently.

Personally, I just rely on `autoLayout` and `.gitignore` the `workspace.json` generated by Lite. If you really need manual layout with multiple contributors, you should use Structurizr's Cloud/On-Prem services — they do lots of [clever](https://docs.structurizr.com/cloud/workspace-locking) [things](https://docs.structurizr.com/cli/push) to enable it.

## Thanks

- [@simonbrowndotje](https://github.com/simonbrowndotje) for the C4 model, and for open sourcing [structurizr/ui](https://github.com/structurizr/ui) with plenty of examples so I could build a diagram renderer without writing any diagram rendering code
