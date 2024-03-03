# AnsibleForJenkins

## ansible.Jenkinsfile pipeline structure

1. `Checkout` - clears workspace and pulls files from repository.

2. `GenerateInventoryAndApply` - generates inventory.ini file and applies playbook:
- `getPublicIpAddress(tag)` - used to retrieve public ip adress by `app-instance` tag
