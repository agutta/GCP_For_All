## Organization

```bash
gcloud organizations list

# for a single org, get its id
ORG_ID=$(gcloud organizations list --format 'value(ID)')

# list top level projects
gcloud projects list --filter "parent.id=$ORG_ID AND  parent.type=organization"

# list top level folders
gcloud resource-manager folders list --organization=$ORG_ID
# list sub folders given upper level folder id
gcloud resource-manager folders list --folder=$FOLDER_ID
# get iam policy for the folder
gcloud resource-manager folders get-iam-policy $FOLDER_ID

# grant roles to a user
ORGANIZATION_ADMIN_ADDRESS='user:developer1@example.com'
gcloud resource-manager folders add-iam-policy-binding ${folder_id} \
  --member=${ORGANIZATION_ADMIN_ADDRESS} \
  --role=roles/resourcemanager.folderAdmin
gcloud resource-manager folders add-iam-policy-binding ${folder_id} \
  --member=${ORGANIZATION_ADMIN_ADDRESS} \
  --role=roles/storage.admin
gcloud resource-manager folders add-iam-policy-binding ${folder_id} \
  --member=${ORGANIZATION_ADMIN_ADDRESS} \
  --role=roles/billing.projectManager

# Organization policies
#List of org policies set at org level
gcloud resource-manager org-policies list --organization $ORG_ID

# List all available constraints in addition to set Organization Policies associated with organization
gcloud resource-manager org-policies list --organization $ORG_ID --show-unset

# List just the name of the constraint changes
gcloud resource-manager org-policies list --organization $ORG_ID --format='value(constraint.basename())'

# Use describe with a constraint name to see the values set for the constraint
gcloud resource-manager org-policies describe constraints/gcp.resourceLocations --organization $ORG_ID

# Lists all constraints set under org and values set for the constraints
for constraint in $(gcloud resource-manager org-policies list --organization $ORG_ID --format='value(constraint.basename())') 
  do
     gcloud  resource-manager org-policies describe $constraint --organization $ORG_ID 
  done

# Lists all constraints in all folders under an org
for folder in $(gcloud resource-manager folders list --organization $ORG_ID  --format='value(name)')
do
    export FOLDER=$folder
    echo "----------------------------------------------"
    echo "Getting Org policies for - " $FOLDER
    for constraint in $(gcloud resource-manager org-policies list --folder=$FOLDER  --format='value(constraint.basename())') 
    do
        gcloud  resource-manager org-policies describe $constraint --folder=$FOLDER 
    done
    echo "----------------------------------------------"
done

# Lists all constraints in all projects under an org. (cloudasset.googleapis.com API needs to be enabled in the project that is set on cloud-shell)
for project in $(gcloud asset search-all-resources --scope organizations/$ORG_ID --asset-types='cloudresourcemanager.googleapis.com/Project' --format='value(name.basename())')
    do
        echo "----------------------------------------------"
        echo "Getting Org policies for - " $project
        for constraint in $(gcloud resource-manager org-policies list --project=$project  --format='value(constraint.basename())') 
        do
            gcloud alpha resource-manager org-policies describe $constraint --project=$project --effective
        done
        echo "----------------------------------------------"
    done
```
