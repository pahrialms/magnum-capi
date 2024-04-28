# How the autoscalling work ? 

## Scale out
Magnum will detect if there's any pod in pending status, if detected magnum will execute to adding more worker node. The pending state that triggers autoscalling is usually due to an increase in the number of replications.

![image](https://github.com/pahrialms/magnum-capi/assets/82088448/25a19999-d219-4d88-9bde-d5df6a747ffa)

## Scale down
Process scale down will occur if there is a reduction in the number of replications
