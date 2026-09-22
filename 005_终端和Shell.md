# 终端和Shell

## 终端

查看Shell类型：echo $SHELL

&nbsp;

基础命令：

- pwd
- ls -la
- clear
- scp model.pt user@gpu-box-ip:~/models/    # Copy files to remote
- scp user@gpu-box-ip:~/results/metrics.json ./    # Copy files from remote
- - rsync -avz ./data/ user@gpu-box-ip:~/data/    # Sync a whole directory 
- diff <(grep "accuracy" exp1.log) <(grep "accuracy" exp2.log)    # Compare two experiment logs side by side
- wget https://huggingface.co/model/resolve/main/model.safetensors    # Download a model from Hugging Face
- tar xzf dataset.tar.gz -C ./data/    # Untar a dataset
- df -h    # Check disk space 
- du -sh ./data/*
- env | grep -i cuda    # Environment variable check before training
- env | grep -i torch
- ssh user@gpu-box-ip    # Basic connection 远程服务器
- ssh -i ~/.ssh/my_gpu_key user@gpu-box-ip    # With a specific key
- ssh -L 8888:localhost:8888 user@gpu-box-ip    # Port forward (access remote Jupyter/TensorBoard locally)

&nbsp;

## 管道

- python train.py > output.log 2> errors.log    # Redirect stdout and stderr to separate files
- python train.py > train_full.log 2>&1    # Redirect both to the same file
- cat train.log | grep "Loss" | wc -l    # Count how many times "loss" appears in a log
- grep "Loss:" train.log | awk '{print $NF}' > losses.txt    # Extract just the loss values from training output
- tail -f train.log | grep --line-buffered "ERROR"    # Watch a log file update in real time, filtering for errors
- grep "final_accuracy" results/*.log | sort -t= -k2 -n -r    # Sort experiments by final accuracy
  - -t=    指定字段分隔符为 =
  - -k2    按第 2 个字段排序
  - -n    按数值排序
  - -r    倒序
- python train.py 2>&1 | tee train.log; echo "DONE" | mail -s "Training complete" you@email.com    # Run training, log everything, notify when done
- find . -name "*.pt" -o -name "*.safetensors" | xargs du -h | sort -rh | head -20    # Find the largest model files
- find . -name "*.py" | xargs wc -l | tail -1    # Count lines in all Python files (see how big your project is)

&nbsp;

## 后台运行

- python train.py &    # Run in background (output still goes to terminal)
- nohup python train.py > train.log 2>&1 &    # Run in background, immune to hangup (closing terminal won't kill it)
- jobs    # Check what's running in background
- ps aux | grep train.py    查看进程
- fg %1    # Bring a background job to foreground
- kill %1    # 杀进程
- kill $(pgrep -f "train.py")    # 杀进程

&nbsp;

任何超过几分钟的时间,使用tmux。

|                             |                          |                     |
| --------------------------- | ------------------------ | ------------------- |
| Method                      | Survives terminal close? | Can reattach?       |
| command &                   | No                       | No                  |
| nohup command &             | Yes                      | No (check log file) |
| screen<br><br>/<br><br>tmux | Yes                      | Yes                 |

&nbsp;

## tmux

tmux（Terminal Multiplexer）是一个终端复用器，能在一个终端窗口里管理多个会话、窗口和面板，并且断开后程序继续在后台运行。

sudo apt install tmux # 安装

- tmux new -s training    # Start a named session
- Ctrl+B then "    # 水平拆分窗口
- Ctrl+B then %    # 垂直拆分窗口
- Ctrl+B then arrow keys    # 切换窗口
- Ctrl+B then d    # Detach (session keeps running)
- tmux attach -t training    # Reattach
- tmux ls    # List sessions
- tmux kill-session -t training      # Kill a session
  
  &nbsp;

典型的人工智能工作流程：

```
tmux new -s train

# Pane 1: start training
python train.py --epochs 100 --lr 1e-4

# Ctrl+B, " to split, then run GPU monitor
watch -n1 nvidia-smi

# Ctrl+B, % to split vertically, tail the logs
tail -f logs/experiment.log

# Now detach with Ctrl+B, d
# SSH out, go get coffee, come back
# tmux attach -t train
```

&nbsp;

## 监测

- htop    # System processes (better than top)    sudo snap install htop
- nvtop    # GPU processes (if you have NVIDIA GPU)    sudo snap install nvtop
- nvidia-smi    # Quick GPU check without nvtop
- watch -n1 nvidia-smi    # Watch GPU usage update every second
- nvidia-smi --query-compute-apps=pid,name,used_memory --format=csv    # See which processes are using the GPU

&nbsp;

## 创建常用别名

- alias gpu='nvidia-smi --query-gpu=index,name,utilization.gpu,memory.used,memory.total,temperature.gpu --format=csv,noheader'    # GPU status at a glance

- alias killtraining='pkill -f "python.*train"'    # Kill all Python training processes

- alias ae='source .venv/bin/activate'    # Quick virtual environment activate

- alias watchloss='tail -f logs/*.log | grep --line-buffered "loss"'    # Watch training loss
