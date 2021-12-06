# How to create branch

Local :

`````markdown
//Without switching current branch.
$git branch newBranchName

//Create new feature with switching current branch.
$git checkout -b newBranchName
`````

Remote :

`````markdown
$git push origin newBranchName
`````



### Connecting between localBranch and remoteBranch

`````mark
$git branch --set-updtream-to origin/newBranchName
//Now the local branch newBranchName is tracking branch of origin/newBranch.
`````

