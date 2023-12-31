# az-batch-scheduler
Run Spark and non-Spark jobs with Azure Batch

# Steps for running jobs without scheduling (Ubuntu + Bash)
The steps assume that you already have an ACR and a Batch account setup in Azure

1. Create all the necesssary task files inside the tasks folder (see example files).

2. Generate an unique job id from command line : 
```
JOB_ID=$(uuidgen)
```

3. Build and push the base docker image (in order to change the image name edit the file containerize_batch_base.sh) : 
```
./containerize_batch_base.sh \
    --registry acr_registry_username.azurecr.io
```

3. Build and push the base docker image (in order to change the image name edit the file containerize_batch_base.sh and containerize_batch.sh) : 
```
./containerize_batch.sh \
    --registry acr_registry_username.azurecr.io \
    --image task_acr_image_name \
    --dockerpath Dockerfile.batch
```

4. Create task(s) : 
```
python3 create_tasks.py \
    --job_id $JOB_ID \
    --app_id APP_ID \
    --tenant_id TENANT_ID \
    --app_secret APP_SECRET \
    --blob_storage_url BLOB_STORAGE_URL \
    --blob_storage_container BLOB_INPUT_CONTAINER
```

5. Run batch :
```
python3 run_batch.py \
    --job_id $JOB_ID \
    --batch_key BATCH_ACCOUNT_KEY \
    --acr_user ACR_USERNAME \
    --acr_pwd ACR_PASSWORD \
    --app_secret APP_SECRET \
    --tenant_id TENANT_ID \
    --app_id APP_ID \
    --pool_id POOL_ID \
    --batch_url BATCH_ACCOUNT_URL \
    --vm_size VM_SIZE \
    --pool_count POOL_SIZE \
    --image TASK_IMAGE_NAME \
    --acr_url ACR_ACCOUNT_URL \
    --num_tasks NUM_TASKS \
    --blob_storage_url BLOB_STORAGE_URL \
    --blob_storage_inp_container BLOB_INPUT_CONTAINER \
    --blob_storage_out_container BLOB_OUTPUT_CONTAINER
```

E.g.

```
python3 run_batch.py \
    --job_id $JOB_ID \
    --batch_key "xxx111" \
    --acr_user "elon" \
    --acr_pwd "elon123" \
    --app_secret "yyy0001" \
    --app_id "1234" \
    --tenant_id "abc-123" \
    --pool_id "sorting-batch" \
    --batch_url "elon.eastus.batch.azure.com" \
    --vm_size "Standard_D4s_v3" \
    --pool_count 10 \
    --image "elon.azurecr.io/azure-batch-demo" \
    --acr_url "elon.azurecr.io" \
    --num_tasks 10 \
    --blob_storage_url "elonstorage.blob.core.windows.net" \
    --blob_storage_inp_container "inputdata" \
    --blob_storage_out_container "outputdata"
```

# Steps for running jobs with scheduling (Ubuntu + Bash)
The steps assume that you already have an ACR and a Batch account setup in Azure

1. Create all the necesssary task files inside the tasks folder (see example files).

2. Generate an unique job id from command line : 
```
JOB_ID=$(uuidgen)
```

3. Build and push the base docker image (in order to change the image name edit the file containerize_batch_base.sh and containerize_batch.sh) : 
```
./containerize_batch_base.sh \
    --registry acr_registry_username.azurecr.io
```

3. Build and push the docker image for your task : 
```
./containerize_batch.sh \
    --registry acr_registry_username.azurecr.io \
    --image task_acr_image_name \
    --dockerpath Dockerfile.scheduler.batch
```

4. Run scheduler for batch :
```
python3 run_scheduler_batch.py \
    --batch_key BATCH_ACCOUNT_KEY \
    --acr_user ACR_USERNAME \
    --acr_pwd ACR_PASSWORD \
    --app_secret APP_SECRET \
    --app_id APP_ID \
    --tenant_id TENANT_ID \
    --pool_id POOL_ID \
    --batch_url BATCH_ACCOUNT_URL \
    --s_vm_size SCHEDULER_VM_SIZE \
    --vm_size TASK_POOL_VM_SIZE \
    --s_pool_count SCHEDULER_POOL_SIZE \
    --image TASK_IMAGE_NAME \
    --acr_url ACR_ACCOUNT_URL \
    --num_tasks NUM_TASKS \
    --blob_storage_url BLOB_STORAGE_URL \
    --blob_storage_inp_container BLOB_INPUT_CONTAINER \
    --blob_storage_out_container BLOB_OUTPUT_CONTAINER \
    --job_schedule_id JOB_SCHEDULE_ID \
    --job_interval JOB_INTERVAL_IN_SECS
```

E.g.

```
python3 run_scheduler_batch.py \
    --batch_key "xxx111" \
    --acr_user "elon" \
    --acr_pwd "elon123" \
    --app_secret "yyy0001" \
    --app_id "1234" \
    --tenant_id "abc-123" \
    --pool_id "sorting-batch" \
    --batch_url "elon.eastus.batch.azure.com" \
    --s_vm_size "Standard_D4s_v3" \
    --vm_size "Standard_D4s_v3" \
    --s_pool_count 1 \
    --image "elon.azurecr.io/azure-batch-scheduler-demo" \
    --acr_url "elon.azurecr.io" \
    --num_tasks 10 \
    --blob_storage_url "elonstorage.blob.core.windows.net" \
    --blob_storage_inp_container "inputdata" \
    --blob_storage_out_container "outputdata" \
    --job_schedule_id "job-scheduler-new" \
    --job_interval 60
```
