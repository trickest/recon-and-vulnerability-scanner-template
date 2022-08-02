<h1 align="center">Recon And Vulnerability Scanner Template
 <a href="https://twitter.com/intent/tweet?text=Recon%20And%20Vulnerability%20Scanner%20Template%20-%20Create%20your%20own%20recon%20%26%20vulnerability%20scanner%20with%20Trickest%20and%20GitHub%20https://github.com/trickest/recon-and-vulnerability-scanner-template&hashtags=automation,asm,cve,recon"><img src="https://img.shields.io/badge/Tweet--lightgrey?logo=twitter&style=social" alt="Tweet" height="20"/></a></h1>
<h3 align="center">Create your own recon & vulnerability scanner with Trickest and GitHub
</h3>

## Repository Structure

- `config.yaml` - Config file for [trickest-cli](https://github.com/trickest/trickest-cli) (**Set the repository name initially**)
- `domains.txt` - List of root domains  (**tld domains list (example.com)**)
- `hostnames.txt` - List of hostnames found for root domains provided (**Updated by the workflow, if updated manually, will be propagated through the entire workflow**)
- `servers.txt` - List of available servers for found hostnames (**Web servers found from hostnames**)
- `reports.txt` - List of vulnerabilities found for found servers (**Vulnerabilities found**)
- `blacklist.txt` - List of strings to exclude from all results (**Blacklist hostnames and servers `grep -vFf`**)
- `templates` (folder) - Place where you push [nuclei](https://github.com/projectdiscovery/nuclei) templates (**Folders supported**)


## Setting Up Configuration

#### 1. Trickest Token & GitHub Deploy Key

Set up the `TRICKEST_TOKEN` variable to the secrets.

- Create a new repository from the [template](https://github.com/trickest/recon-and-vulnerability-scanner-template)
- Open [https://github.com/YOUR_USERNAME/YOUR_REPOSITORY/settings/secrets/actions](#)
- Add `TRICKEST_TOKEN`, which can be found at [https://trickest.io/dashboard/settings/my-account](https://trickest.io/dashboard/settings/my-account)

Set up a [GitHub deploy key](https://docs.github.com/en/developers/overview/managing-deploy-keys#deploy-keys) with write access to your Bug Bounty Setup repository and add the private SSH key to the `SSH_KEY` action secret.

![GitHub action secrets](/images/github-action-secrets.png)


#### 2. Config File

Replace `REPOSITORY_NAME`  with your GitHub repository name inside the [config.yaml](config.yaml) file.

```
inputs:   
  string-to-file-1.string: REPOSITORY_NAME
  recursively-cat-all-5:
    file:
      - id_rsa
machines:
  large: 1
```

#### 3. Root Domains

All of the domains will be picked up automatically by the workflow. You will need to push the new root domain names to the `domains.txt` file.

```
echo "trickest.com" > domains.txt
```

#### 4. Nuclei Templates

All of the nuclei templates will be picked up automatically by the workflow. Push the new nuclei templates to the `templates` folder.

```
cd templates
wget "https://raw.githubusercontent.com/projectdiscovery/nuclei-templates/master/cves/2022/CVE-2022-35416.yaml"
```

#### 5. Pushing the data
When you're done adding your data/templates, commit and push 

```
git add *
git commit -m "Add target"
git push
```

#### 5. Running the workflow

The workflow is triggered on `workflow_dispatch` event; feel free to change the trigger the way it suits the best your use case (The `push` event might be a suitable option if you want to trigger the workflow automatically).

![Workflow Dispatch Event](/images/workflow-dispatch-event.png)



## Workflow Building

#### 1. Workflow Repository Setup

Initially, you need to gather the data about the repository:

- String inputs are colored purple, and you need to connect your repository name and email to the `string-to-file` (this file will be used as a variable value when cloning the repository inside of `get-repo-data`)
- `id_rsa` is directly uploaded through the client/workflow and is used when cloning the repository

![GitHub Repository Data](/images/github-repository-data.png)

> NOTE: Keep in mind that out/output.txt is reserved for the output file port, so you can cat the content of `domains.txt` to `out/output.txt` to be available for `Amass` and `SubFinder`.
> 

#### 2. Passive Recon

Now that you have your root domains, you should use `SubFinder` and `Amass` to get all of the results from passive sources. Their outputs will be merged through `recursively-cat-all-1` with additional `sort -n | uniq`, which will cat and deduplicate your results.

![Passive Recon](/images/passive-recon.png)

#### 3. Active Recon

This part consists of getting all the results from Passive Recon and then:

- Executing [dsieve](https://github.com/trickest/dsieve) with the `2:4` flag for subdomain levels to get all of the environments
- Executing [mksub](https://github.com/trickest/mksub) to create a wordlist of potential hostnames
- Resolving with [puredns](https://github.com/d3mondev/puredns)
- Executing found brute-forced hostnames permutations with [gotator](https://github.com/Josue87/gotator)
- Resolving gotator with [puredns](https://github.com/d3mondev/puredns)

![Active Recon](/images/active-recon.png)


#### 4. Servers and Scan

Finally, you've got your hostnames for this run, and now you can pass it to [httpx](https://github.com/projectdiscovery/httpx) to get all of the web servers. `get-repo-data` node will provide you with the [nuclei](https://github.com/projectdiscovery/nuclei) templates you already pushed to the repository.

![Servers & Scanners](/images/servers-scanners.png)


#### 5. Push The Results

Finally, the `update` node will get the data and push it to the repository.

Integrations are a crucial part of Attack Surface Management, check out [our post](https://trickest.com/blog/picking-attack-surface-management-solution/#integrations) on how to pick the right one.

## Did somebody say diffs?

Commit messages will show what is changed, which means you will have an insight into all the new data (vulnerabilities). How awesome is that!

![GitHub Diffs](/images/diffs.png)


[<img src="./banner.png" />](https://trickest-access.paperform.co/)
