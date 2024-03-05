# Deploy Azure Open AI Java reference template on Azure Container Apps production ready environment using ACA LZA

## Deployment Architecture

![Deployment Architecture](aca-internal-java-ai.png)

## Getting Started
### Deploy the infrastructure
1. Clone the ACA LZA Java App accelerator repo. `git clone --single-branch --branch aca-lza https://github.com/dantelmomsft/chat-with-your-data-java-lza-app-accelerator.git`
2. Run `cd chat-with-your-data-java-lza-app-accelerator/infra/aca` 
3. Review the bicep parameters in `bicep/chat-with-your-data-java-aca-main.parameters.json`.Pay attention to vmLinuxSshAuthorizedKeys param: you should provide here the a public key that you have generated along with your private key. For more information on how to generate a public key see [here](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/create-ssh-keys-detailed).
4. Run `azd auth login` to authenticate with your Azure subscription.
5. Run `azd provision` to provision the infrastructure and when asked by the prompt provide an env name and the deployment region.This will take several minutes (about 30/40 min) and will:
    - Download the ACA lza code in the folder `infra/aca/bicep/lza-libs`.
    - Automatically run the ACA lza bicep source code.
    - Automatically run the java app bicep source code in the folder `chat-with-your-data-java-lza-app-accelerator\infra\aca\bicep\modules`. This will create the Azure supporting services (Azure AI Search, Azure Document Intelligence, Azure Storage, Azure Event Grid, Azure Service Bus) required by the app to work following the best practices provided by the ACA LZA infrastructure.
    -  Automatically create `.azure` folder with azd env configuration. you should see a folder like this: `chat-with-your-data-java-lza-app-accelerator\infra\aca\.azure`
### Deploy the Java app
1. Connect to the jumpbox using bastion, and run `git clone https://github.com/dantelmomsft/chat-with-your-data-java-lza-app-accelerator.git`
   - You can use the bastion from the Azure portal to connect to the jumpbox. Info [here](https://learn.microsoft.com/en-us/azure/bastion/bastion-connect-vm-ssh-linux)
   - You can use a ssh native client command from your local terminal. Info [here](https://learn.microsoft.com/en-us/azure/bastion/connect-vm-native-client-windows#connect-linux)
2. Run `cd chat-with-your-data-java-lza-app-accelerator` 
3. To download the the chat-with-your-data-java [source code ](https://github.com/Azure-Samples/azure-search-openai-demo-java) run:
    - *Windows Power Shell* - `.\scripts\download-app-source.ps1 -branch main` 
    - *Linux/Windows WSL* - `./scripts/download-app-source.sh --branch main`.
4. Run `cd chat-with-your-data-java-lza-app-accelerator/infra/aca` and copy here the `chat-with-your-data-java-lza-app-accelerator\infra\aca\.azure` local folder that has been created on your laptop at the end of [Deploy Infrastructure](#deploy-the-infrastructure) phase.
    - You can use a scp command from your local terminal. Info [here](https://learn.microsoft.com/en-us/azure/bastion/vm-upload-download-native#tunnel-command)    
5. Run `azd auth login`
6. run `azd deploy`. This will build and deploy the java app.
7. From your local browser connect to the public azure application gateway using https. To retrieve the App Gateway public IP address, go to the Azure portal and search for the application gateway resource in the spoke resource group. In the overview page copy the "Frontend public IP address" and paste it in your browser.
![App Gateway Public IP address](app-gateway.png)

### Ingest the predefined documents
Once you deployed the app you can ingest the predefined documents in the data folder. You can use the following steps:
1. Connect to the jumpbox using bastion as described in the previous section.
2. Run `cd chat-with-your-data-java-lza-app-accelerator/infra/aca`
3. Run `./scripts/prepdocs.sh`

### Known Issues and gaps
- Putting the content of your id-rsa.pub file containing the public key in the bicep params files it's not working. As a workaround you can setup it using the following command: `az vm user update --resource-group {spoke resource group name} --name {vm name} --username azureuser --ssh-key-value ~/.ssh/id_rsa.pub`
- Using the "Microsoft.Compute/virtualMachines/runCommands" bicep resource to automatically setup jumpbox VM with tools dependencies might not work.As a workaround, while you are on connected to the jumpbox, you can run manually the script in the `chat-with-your-data-java-lza-app-accelerator\infra\aca\bicep\modules\vm\jumpbox-tools-setup.sh` file.