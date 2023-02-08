# Logseq Publish Action

Publish your [Logseq](http://github.com/logseq/logseq) graph with a GitHub Action. ✨

This action is the missing piece for achieving a complete CD workflow for your public Logseq graph.

- It will load your graph and publish it into `www` (this is configurable) folder.
- Next, you can then deploy `www` folder with [JamesIves/github-pages-deploy-action](https://github.com/JamesIves/github-pages-deploy-action).

## Example Logseq Graphs Published with this Action

- [Logseq documentation](https://docs.logseq.com/)
- [Xuanwo's Note](https://note.xuanwo.io/)
- [Knowledge Garden](https://pengx17.github.io/knowledge-garden/)
- [Wiki of reuixiy!](https://io-oi.me/wiki/)
- [Zhicheng's docs](https://docs.unickcheng.cc/)

## Usage

A [video](https://www.youtube.com/watch?v=UYqJcFEYUsY) to walk through.

Firstly, add this step to your GitHub workflow. You can refer to My Example Graph's [workflows/main.yml](https://github.com/pengx17/knowledge-garden/blob/main/.github/workflows/main.yml) as an example.

```yml
steps:
  - uses: actions/checkout@v2
  - name: Logseq Publish 🚩
    uses: pengx17/logseq-publish@v0.1
  - name: add a nojekyll file # to make sure asset paths are correctly identified
    run: touch $GITHUB_WORKSPACE/www/.nojekyll
  - name: Deploy 🚀
    uses: JamesIves/github-pages-deploy-action@4.1.9
    with:
      branch: gh-pages # The branch the action should deploy to.
      folder: www # The folder the action should deploy.
      clean: true
```

Whenever you push changes to your GitHub repo, your graph will be published to the `gh-pages` branch. A few minutes later, your GitHub Pages will get updated.

## All options

- `src`: Publish graph src directory. Defaults to `` (empty string, root repo directory).
- `dest`: Publish destination directory. Defaults to `www`.
- `trace`: Whether to generate and publish trace file for debugging. This trace file will be uploaded as an artifact to the run. Defaults to `true`.
- `theme`: Selected theme. Defaults to `light`.

## How it works

Here is a [document](https://pengx17.github.io/knowledge-garden/#/page/logseq%20publish%20github%20action) about the story behind this action.

TLDR., this action will start Logseq desktop App in a Docker container, automate it by Playwright and finally load & publish the graph.

Since most of the work is done in a [prepared Docker container](https://github.com/pengx17/logseq-publish/pkgs/container/logseq-publish), you can refer to [action.yml](./action.yml) and adapt it in other places without the need for GitHub actions.

Hint: you can checkout the trace file in the action artifacts and view it in the [Playwright trace viewer](https://trace.playwright.dev/) to see what's going on in the action.

https://user-images.githubusercontent.com/584378/163351229-ba281c08-c29d-4cf5-8223-337bf80b8726.mp4

## Alternate usage examples

- One-liner Powershell call with only Docker installed as prerequisite.
  Below assumes that your folder workfolder has "KnowledgeGraph" for the graph, and "HTMLOutput" for the output. Container will be created and discarded on the fly.

```
PS C:\> docker run -d -i  `
	     --name LogSeqPublishContainer  `
	     --rm `
	     -v D:\WorfklowScripts\PublishLogseqExport\graph:/home/logseq/graph  `
	     -w /home/logseq  `
	     ghcr.io/pengx17/logseq-base:master  `
	     bash -c "xvfb-run node /home/logseq/graph/publish.mjs -p /home/logseq/graph/KnowledgeGraph -t /home/logseq/graph/build_trace.txt -o /home/logseq/graph/HTMLOutput > /home/logseq/graph/build.log"
```

- Use logseq-publish on gitlab: [@MarkusPic](https://gist.github.com/MarkusPic) provides an example based on gitlab CI [.gitlab-ci.yml](https://gist.github.com/MarkusPic/a757e52b2be8013161165483ad9bf4ed)

## Local development

- clone https://github.com/logseq/logseq.git & https://github.com/pengx17/logseq-publish.git
- build logseq app
	- cd into logseq src root
  - change  `"name": "Logseq"` in `resources/package.json` to `"name": "something-else"`
	- run `yarn` && `yarn release`
  - cd into `static` and run `yarn` && `yarn rebuild:better-sqlite3`
- copy logseq static into `logseq-publish/public/static`
	- in logseq-publish root, run
		- `mkdir public`
		- `cp -r ../logseq/static ./public/static` (assuming logseq and logseq-publish is at the same folder)
- install logseq-publish deps
	- makes sure you have `pnpm` & Node > 16 installed
	- run `pnpm i`
- now you should be able to use `node publish.mjs` to run the script

Hints:
- use [Playwright Debugging tools | Playwright](https://playwright.dev/docs/debug) to debug the script. `PWDEBUG=1 node publish.mjs`
