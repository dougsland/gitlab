Example of **.gitlab-ci.yml**
```yaml
image: quay.io/centos/centos:stream9

default:
  tags:
    - docker

stages:
  - test

update_readme_toc:
  stage: test
  before_script:
    - dnf install -y git
  script:
    - find . -name "README.md" -exec .gitlab/generate-TOC/generate-TOC {} \;
    - |
     git diff --exit-code || (
       echo "Error: The Markdown Table of Contents in one or more README.md files is outdated."
       echo "Please update it manually."
       echo "You can use the tool with the following command:"
       echo ""
       echo "  find . -name 'README.md' -exec .gitlab/generate-TOC/generate-TOC {} \;"
       echo ""
       exit 1
     )
  rules:
    - changes:
        paths:
          - "**/README.md"
      when: always
```
