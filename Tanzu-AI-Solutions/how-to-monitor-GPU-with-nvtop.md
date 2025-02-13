# How to monitor GPU with nvtop

- SSH to VMware Operations Manager VM with user ubuntu (make sure the private key that corresponds to the public key specified during the Ops Manager install is added to your workstation)

- Retrieve "bosh commandline credentials" from Tanzu Operations Manager > BOSH Director > Credentials > bosh commandline credentials

- Create bosh alias in ssh session using the env vars retrieved in previous step eg
  - ```alias bosh = "BOSH_CLIENT=ops_manager BOSH_CLIENT_SECRET=i1GXRRESqDk9Iddvovvx_qCA2Lqyxvab BOSH_CA_CERT=/var/tempest/workspaces/default/root_ca_certificate BOSH_ENVIRONMENT=10.0.70.11 bosh "```

- Discover the instance name for the vm which is using your GPU. Identify the vm using "VM Type" which matches what you specified in the GenAI tile for the model
  - ```bosh vms -d genai-models```

- bosh ssh to the instance eg
  - ```bosh ssh -d genai-models 2be069cc-358a-4695-96ab-5beef1579d22/3c7602b1-945b-4333-80b8-25123fe6d749```

- Install nvtop
  - ```sudo apt install nvtop```

- run nvtop
  - ```nvtop```

![nvtop output](/Tanzu-AI-Solutions/assets/nvtop_output.jpg)
