# Quickstart

# Login
`az login --tenant <tenantId>`

`azd auth login --tenant-id <tenantId>`

# Quota
Run `./infra/scripts/quota_check_params.sh --models gpt-4o-mini:50` to make sure which region/model can be used. It will return a list of regions and models that show how many tokens per minute they support and how many are available in the relative recent time span (the numbers returned aren't necesarrily the current executing minute as it can be cached and contain the highest amount in the relative recent history and may not change much if usage is low and appear static).

`globalstandard` is the default deployment type

`Gpt-4o-mini` is the default model.

`eastus` is the default location.

`eastus2` is the default secondary location.

# Env variables
Before running `azd up`, we want some env vars set correctly so the resources get created correctly.

## Default Env
A default environment is required. It can be set indirectly when running other commands. The easiest is to set an environment variable for something we already need to change, and it will prompt you to set the environment.

Update TPM capacity: default capacity is 30 but we need ~50 for a single LLM request so if have capacity, they recommend 150:
`azd env set AZURE_OPEN_AI_DEPLOYMENT_MODEL_CAPACITY 150`

Now you get prompted to set the environment, supply the value `convo-miner` (we dont need to do dev/qa/etc since just demo)

## Optional Env overrides
At this point if you want to use different defaults you can update the values at `.azure/convo-miner/.env` or use the `azd env set` command.

Common modifications: 
- `azd env set AZURE_OPEN_AI_DEPLOYMENT_MODEL gpt-4o`
- `azd env set AZURE_LOCATION eastus2`
- `azd env set AZURE_SECONDARY_LOCATION westus`

# Deploy resources
Everything is set up to be able to run the application as soon as its deployed.

Run `azd up`

When prompted:
- Select an Azure Subscription to use:                      1. <name here>                  
- Select an Azure location to use:                          48. (US) East US (eastus)
- Pick a resource group to use:                             1. Create a new resource group
- Enter a name for the new resource group (rg-convo-miner): <enter>

Takes about 10 minutes

if get validation error requiring resource group tags of owner/purpose then ask JT to remove the policy requiring that...azure.yaml file that azd uses doesnt not support setting tags (though you could create the resource group first manually, set tags then run azd up)

If get this error, ignore as its only console writing the app url

ERROR: error executing step command 'provision': failed running post hooks: 'postprovision' hook failed with exit code: '0', Path: 'C:\Users\TOBYST~1\AppData\Local\Temp\azd-postprovision-1577937485.ps1'. : executable file not found in %PATH%

# Run the app
Go to the region and find the App service whose name is "app-*" and open its URL. The app is ready to go to chat (see DevelopmentGuide.md for example chat questions to ask).

In my tests, with 150k TPM, I was able to ask 3 questions in a minute before hitting rate limit

# Destroy resources
`azd down` 

or 

`azd down --purge` (to permanently delete resources allowing soft delete, like KV/AIService)

*took 27 minutes

If you want to update this repo from the repo forked from, run this:

`git remote add upstream https://github.com/microsoft/Conversation-Knowledge-Mining-Solution-Accelerator.git`

...then can rebase via `git pull upstream main`