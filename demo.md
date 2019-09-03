#RHTE Demo

1. Setup the cluster
1. Get a Che workspace ready
1. Open git repo: `https://github.com/bmicklea/budgetlist` and copy link into clipboard
1. Click factory link
1. Describe what CRW is doing
1. Switch to pre-created workspace
1. Notice CVEm, explain and fix
1. Git commit / push
1. Explain how the app will be used - good candidate for serverless
1. Add to topology from git - use pasted Git URL
1. Name `bl-serverless`
1. Select serverless option and scaling advanced options (limit to 2 pod)
1. Switch to Builds in dashboard
1. Select `bl-serverless`
1. Go to "Builds" and select `bl-serverless-1`
1. If needed go to Logs
1. When build completes go to topology
1. Open details panel for `bl-serverless`
1. Actions > Edit Annotations
1. Add `app.openshift.io/connects-to` and `mongodb`
1. 
