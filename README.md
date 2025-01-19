## This is ChatAI docker!


### ENABLE NVIDIA GPU ACCELERATION

```
sudo apt-get install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

### DISABLE NVIDIA GPU ACCELERATION

Comment out the ``` runtime: nvidia ``` line from ``` docker-compose.yml ```
