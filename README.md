echo ">>> Step 0: ensure pool exists"
az batch pool show \
  --pool-id $(PoolId) \
  --account-name $(BatchAccount) \
  --account-endpoint https://$(BatchAccount).$(Region).batch.azure.com \
  && echo "Pool exists" \
  || az batch pool create \
        --id $(PoolId) \
        --vm-size $(VmSize) \
        --target-dedicated-nodes 1 \
        --image $(ImagePublisher):$(ImageOffer):$(ImageSku) \
        --account-name $(BatchAccount) \
        --account-endpoint https://$(BatchAccount).$(Region).batch.azure.com \
        --start-task-command-line "/bin/echo pool ready"
