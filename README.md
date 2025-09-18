## How to use ?

```yml
      - name: Deploy using Teleport
        uses: anilrajrimal1/deploy-with-teleport@master
        with:
          TELEPORT_HOST: ${{ env.TELEPORT_HOST }}
          TELEPORT_TOKEN: ${{ secrets.TELEPORT_TOKEN }}
          SSH_USER: ${{ steps.get_vm_conf.outputs.SSH_USER }}
          SSH_HOSTNAME: ${{ steps.get_vm_conf.outputs.SSH_HOSTNAME }}
          BRANCH_NAME: ${{ steps.extract_branch.outputs.branch }}
          PROJECT_DIR: ${{ steps.get_vm_conf.outputs.PROJECT_DIR}}
          COMPOSE_FILE_PATH: "./docker/server/"
          PHASE_SERVICE_TOKEN: ${{ secrets.PHASE_SERVICE_TOKEN }}
          PHASE_APP_ID: ${{ secrets.PHASE_APP_ID }}
          PHASE_HOST: ${{ env.PHASE_HOST }}
          PHASE_ENV: ${{ steps.get_vm_conf.outputs.PHASE_ENV }}
          ENV_FILE: "."
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_REGION: ${{ env.AWS_REGION }}
          ECR_REGISTRY: ${{ env.ECR_REGISTRY }}
```