# Lesson 3: Using Git

- [Lesson 3: Using Git](#lesson-3-using-git)
    - [3.1 Understanding Git](#31-understanding-git)
      - [Understanding Git](#understanding-git)
      - [Working with Git](#working-with-git)
      - [Understanding Git Platforms](#understanding-git-platforms)
      - [Using an Existing Git Repository](#using-an-existing-git-repository)
      - [Understanding Git Workflow](#understanding-git-workflow)
      - [Understanding Git Trees](#understanding-git-trees)
      - [Using Git Trees](#using-git-trees)
      - [Understanding File States](#understanding-file-states)
    - [3.2 Git Fundamentals](#32-git-fundamentals)
      - [Setting GiT Client Options](#setting-git-client-options)
      - [Working with Git](#working-with-git-1)
      - [Pushing Files](#pushing-files)
      - [Demo: Using Git](#demo-using-git)
    - [3.3 Using Git Advanced Authentication](#33-using-git-advanced-authentication)
      - [Using Tokens](#using-tokens)
      - [Creating a Credentials Cache](#creating-a-credentials-cache)
      - [Demo: Creating a Credentials Cache](#demo-creating-a-credentials-cache)
    - [3.4 Working with Branches and Merges](#34-working-with-branches-and-merges)
      - [Understanding Branches](#understanding-branches)
      - [Comparing Branches](#comparing-branches)
      - [Using Branches](#using-branches)
      - [Understanding Merges](#understanding-merges)
      - [Demo: Using Branches](#demo-using-branches)
    - [3.5 Organizing Git Repositories for GitOps Environments](#35-organizing-git-repositories-for-gitops-environments)
      - [Understanding the Challenge](#understanding-the-challenge)
      - [Understanding Single Branch Strategy](#understanding-single-branch-strategy)
      - [Understanding Multi-branch Strategy](#understanding-multi-branch-strategy)
      - [Managing Configuration](#managing-configuration)

### 3.1 Understanding Git

#### Understanding Git

- Git is a distributed version control system.
- Developers can work in projects together, where each file revision is tored in the system.
- Old versions of files are always available, and a log is maintained of who made which change to which file.
- Git is distributed, because it allows developers to work offline after cloning a remote repository localy.

#### Working with Git

- Git works with a repository, which can contain different development branches.
- Developers and users can easily upload as well as download new files to and from the Git repository.
- To do so, a Git client is needed.
- Git clients are available for all operating systems.

#### Understanding Git Platforms

- GitHub is a common platform.
- GitLab is a common alternative.
- Private Git servers can also be hosted.
- This course uses GitHub as the standard platform.

#### Using an Existing Git Repository

- **git clone**: creates a linked copy in a newly created directory which will continue to synchronize.
- **git pull**: incorporates changes from a remote repository in the current branch. During this process, diverging branches will be reconciled.
  - Use this to update the current local copy.
- **git fetch**: retrieves the latest metadata from the original branch without fetching changed files.
  - Use this for checking before actually pulling, **git diff** .. **origin** shows changes.
- **git fork**: creates an independent copy of the original Git repository in your GitHub account. Use the Fork option that shows on GitHub for each repository.

#### Understanding Git Workflow

- After cloning, the developer makes changes to the working tree.
- All changes are made to the local repository.
- Developers can push changes to the remote repository.
- If the local repository is network accessible, the owner of the repository can pull changes from it.

#### Understanding Git Trees

- A Git repository consists of three trees maintained by Git.
- The trees are stored as files in the **.git** directory.
- The working directory holds the actual files.
- The indes acts as a staging area.
- The HEAD points to the last commit that was made.

#### Using Git Trees

- When working with Git, the **git add** command is used to add files to the index.
- To commit these files to the head, use **git commit -m 'commit message'** 
- Use **git add origin https://server/reponame** to connect to the remote repository.
- To complete the sequence, use **git push origin main**. Replace "main" with the actual branch you want to push changes to.

#### Understanding File States

- The **git status** command can be used to req uest current file state.
- Files can be in one of three different states.
  - **Modified**: the file in the working tree is different from the file in the repository.
  - **Staged**: the modified file was added to a list of changed files in the index.
  - **Committed**: the modified file has been committed to the HEAD and is ready to be pushed.
- After committing a file to the local repository, it can be pushed to the remote repository.

### 3.2 Git Fundamentals

#### Setting GiT Client Options

- Before using a Git client, you can set options.
- Global options are written to the ~/.gitconfig file.
- Options specific to a repository are stored in the .git directory.
- Use `git config -l --global` to display a list of global options.
- Set your user information.
  - git config --global user.name "Your Name"
  - git config --global user.email "your@email.com"
  - To set passwords, you'll need to use the Git client and setup a credentials cache.
- To set options on a project-only basis, use **git config** without **--global**

#### Working with Git

- Working with Git starts on the Git server, where the upstream repository is created.
- Next, a directory on the local workstation is created.
- **git init** is used to initialie the Git metadata in thw workstation directory.
- Next, a README.md is added, containing information about the contents of the Git directory.
- From there, the normal Git command flow can be applied.
  - **git add*** to add all files to the staging area.
  - **git commit -m 'commit message'** to commit these files to HEAD.

#### Pushing Files

- To push files from the HEAD to the remote repository, use **git push**
- Different push methods are available, use **git config --global push.default** to set the default push method.
  - Use **push.default matching** to push all branches.
  - Use **push.default simple** to push the current branch only (recommended)

#### Demo: Using Git

```bash
# 1. Create a Git repository at https://github.com
# On the client set defaults.
git config --global user.email "your@email.com"
git config --global user.name "Your Name"
git config --global init.defaultBranch main
mkdir gitdemo
cd gitdemo
git init
tree .git
echo "Hello" > README.md
git add *
git commit -m 'initial commit'
git status
git branch -M main
git remote add origin https://github.com/anilitblr/gitdemo
git push -u origin main
```

### 3.3 Using Git Advanced Authentication

#### Using Tokens

- Passwords are relatively insecure, and personal access tokens should be used as an alternative.
- Tokens can be generated with specific permissions.
  - Classic personal access tokens can be set to never expire.
  - Fine-grained access tokens can be configured with access to specific parts of Git.
- Generate a Token from the Git website.
  - Select your account **> Settings**
  - On the left, select **Developer Settings**
  - From here, generate the token
- On the Linux CLI, enter the token when prompted for a password.

#### Creating a Credentials Cache

- Bacause tokens are now the standard way to authenticate, the global username and password settings no longer work for authentication.
- To cache credentials, a credentials cache should be created.
- Use the **gh** client to create this cache.
- Use the personal access token to establish your identity.

#### Demo: Creating a Credentials Cache

- Install the GitHub CLI (see docs.github.com for instructions)
- Use **gh auth login** to create the credentials cache.
- Answer the following prompts.
  - What account do you want to log into?
  - What is your preferred protocol for Git operations?
  - Authenticate Git with your GitHub credentials
  - How would you like to authenticate GitHub CLI

### 3.4 Working with Branches and Merges

#### Understanding Branches

- Branches are used to develop few features in isolation from the main branch.
- The main branch is the default branch, other branches can be manually added.
- Any Git repository can have multiple branches to allow developers to work independently.
- After completion, merge the branches back to the main.

#### Comparing Branches

- Use **git diff <branch1> <branch2>** to preview branch differences and see if there are any potential conflicts.
- To merge another branch in your active branch, use **git merge <branch>**

#### Using Branches

- Use **git branch dev-branch** to create a new branch and start using it.
  - **git checkout -b dev-branch** is the old but deprecated way to do this.
- Use **git push --set-upstream origin dev-branch** to push the new branch to the remote repository.
- Use **git switch main** to switch back to the main.
  - **git checkout main** is the old but deprecated way to do this.
- Delete the branch using **git branch -d dev-branch**

#### Understanding Merges

- After making changes to a brach, the changes can be merged in the main branch.
- First, have a look at the differences: **git diff main dev-branch**
- After using **git checkout main**, use **git merge dev-branch** to apply changes from the branch to the main branch.
- If files have been modified in multiple brnaches, automatic merges are not possible when merge conflicts exist.
- In the case of a merge conflict, the conflicts must be resolved manually by editingthe affected files.

#### Demo: Using Branches

```bash
git branch newfiles
git status
git switch newfiles
echo three > three.txt
git add *; git commit -m 'three'
git push --set-upstream origin newfiles
git switch main
git status
ls # file three does not exist.
git merge newfiles # To merge changes from newfiles in the main brnach.
```

### 3.5 Organizing Git Repositories for GitOps Environments

#### Understanding the Challenge

- In a GitOps Environment, stages are defined for different environments.
- These stages and environments should be presented in the Git repositories.
- Common solutions are the single branch strategy and the multi-branch strategy.

#### Understanding Single Branch Strategy

- In the single branch strategy, one Git repository is used, with a specific directory for each of the environments.
- Kubernetes Kustomie is used to adopt the configuration for a specific environment.
- Environment-specific settings will be in each of the subdirectories, as different resources are required in production and pre-production.
- The advantage of this strategy is that all code is still in one branch, but Kustomiztion is required to create the specific environments.

#### Understanding Multi-branch Strategy

- In a multi-branch strategy, each branch has the specific configuration that is required in an environment.
- As a disadvantage, there is no common code that is shared through all the environments, which is likely to make multi-branch less efficient.

#### Managing Configuration

- Creating configuration for the different environments is taken care of by Kubernetes.
- The core Kubernetes resources to do so are Configmap and Secret.
- Kubernetes provides differentsolutions to manage configurations which will be explored in more depth in Lession 10.
  - Helm  
  - Kustomize
