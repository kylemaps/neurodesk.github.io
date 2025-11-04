_Information on the **Neurodesk** project is available at [neurodesk.org](https://neurodesk.org)_

#### Cloning this repository
Using SSH
```
git clone --recurse-submodules git@github.com:NeuroDesk/neurodesk.github.io.git
```
Using Https:
```
git clone --recurse-submodules https://github.com/NeuroDesk/neurodesk.github.io.git
````

#### Download hugo extended
```
https://github.com/gohugoio/hugo/releases
```

### Forking this repository

#### 1. Find BASEURL

The forked baseurl will usually be in the following format `https://<USERNAME>.github.io/neurodesk.github.io/`. 

#### 2. Set page builder
Go to [/settings/pages](../../settings/pages). Under **Build and deployment**, set **Source** as **Github Actions**

Workflow map (quick)
--------------------
1. Contributor (fork-based)
   - Fork `neurodesk/neurodesk.github.io` → create a branch → open PR against staging `main`.
2. PR checks & preview
   - CI builds PR and publishes preview (Could take 3-5 minutes) at `https://neurodesk.github.io/pr-<PR_NUMBER>/`.
   - Preview url will appear at comment section by a github-actions bot
3. Merge to staging
   - After review, merge PR into staging `main`.
   - Check changes are present at neurodesk.github.io
4. Promote to production
   - Maintainer runs the **Promote to Prod** workflow in `neurodesk/neurodesk.github.io` → Actions or via tagging.
   - Workflow builds site with production baseURL, writes `public/CNAME`, pushes to `neurodesk/neurodesk.org:site`, and triggers deploy.
5. Production deploy
   - `neurodesk/neurodesk.org` deploys from `site` branch.