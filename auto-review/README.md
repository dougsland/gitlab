# 1. Create a GITLAB Token

Log in to GitLab

Click on your profile picture or avatar in the top right corner of the page. Select Settings from the dropdown menu. Access Tokens.

In the Project Tokens section, enter a name for your token in the Name field.

Set an expiration date for the token (optional but recommended).

Name: cicd_bot (the name that will appear when the gitlab will assign people)
Role: Owner

Select the following scopes:
api
read_api
create_runner
read_repository
write_repository
read_registry (optional, if needed)
write_registry (optional, if needed)
k8s_proxy (optional, if needed)
ai_features (optional, if needed)

Click the Create personal access token button.
Save Your Token.

Once the token is created, GitLab will display it. Make sure to copy and save the token immediately. GitLab will not show it again.
Store your token in a secure place, such as a password manager.

# 2. Add GITLAB_TOKEN env variable into CI/CD Settings

## 2.1 Go to Project Settings
Navigate to **Settings** > **CI/CD** in your project.

## 2.2 Expand Variables (Project Settings)
Scroll down to the **Variables** section.

### 2.3 Add/Edit Variables:
Ensure `GITLAB_TOKEN` is added with the correct token.

## 3. Expand Runners (Project Settings)
If your project has at least one runner assigned, you can go to next step. Otherwise, use the **New project runner** button or the **three dots** on the side to **Show runner installation and registration instructions**.

As soon the project has at least **one runner assigned** to go next step.

# 4. Create .gitlab-ci.yml

```sh
git clone git@gitlab.cee.DOMAIN.com:dlandgra/validator-subsystem.git
cd validator-subsystem
mkdir .gitlab
```

Save the following assign_merge_request stage in **.gitlab/.gitlab-ci.yml** 
```yaml
assign_merge_request:
  stage: deploy
  script:
    - dnf install python3-pip -y
    - pip3 install python-gitlab
    - pip3 install requests
    - python3 .gitlab/assign_merge_request
  only:
    - merge_requests
```

# 5. - Create script assign_merge_request

```sh
vi .gitlab/assign_merge_request
```

```python
import os
import gitlab


# Get the GitLab token from environment variables
GITLAB_TOKEN = os.getenv('GITLAB_TOKEN')

# GitLab automatically sets environment variables for each CI/CD job.
#   - CI_PROJECT_ID        -> Project ID
#   - CI_MERGE_REQUEST_IID -> Merge Request ID
PROJECT_ID = os.getenv('CI_PROJECT_ID')
MERGE_REQUEST_IID = os.getenv('CI_MERGE_REQUEST_IID')

# Check if environment variables are available
if not all([GITLAB_TOKEN, PROJECT_ID, MERGE_REQUEST_IID]):
    raise EnvironmentError("Required environment variables are missing")


# Create a GitLab instance
gl = gitlab.Gitlab(
    'https://gitlab.cee.DOMAIN.com',
    private_token=GITLAB_TOKEN,
    ssl_verify=False)  # Install IT certificate in the system and remove this line

# Get the project
project = gl.projects.get(PROJECT_ID)

# Get the merge request
merge_request = project.mergerequests.get(MERGE_REQUEST_IID)

# Define your team assignees
assignees = [
    'USERNAME1',
    'USERNAME2',
]

# Define your team reviewers
reviewers = [
    'USERNAME1',
    'USERNAME2',
]

# Assignees
merge_request.assignee_ids = [
    gl.users.list(username=username)[0].id for username in assignees
]

# Reviewers
merge_request.reviewer_ids = [
    gl.users.list(username=username)[0].id for username in reviewers
]

# Add labels
merge_request.labels += [
    'needreview',
    'podman',
    'systemd'
]

merge_request.save()
```


## Testing

After the gitlab configuration is done, let's test it.

```bash
git clone git@gitlab.cee.DOMAIN.com:dlandgra/validator-subsystem.git
cd validator-subsystem
git checkout -b testbranch
echo "test" >> README.md
git add README.md
git commit -m "this is a test" 
git push --set-upstream origin testbranch

<SNIP message from git>
To create a merge request for testbranch, visit:
remote:   https://gitlab.cee.DOMAIN.com/dlandgra/validador-subsystem/-/merge_requests/new?merge_request%5Bsource_branch%5D=testbranch
```

**PLEASE NOTE**  
No need to set Assignees or Reviewers, just select the Create merge request button. The CI/CD job will automatic set it for you as soon it's done.
