### 问题背景
在autodl上运行rag大作业代码
```
class Generator:
    def __init__(self, model_name='Qwen/Qwen2-7B-Instruct'):
        print(f"Loading generator model: {model_name}...")
        self.tokenizer = AutoTokenizer.from_pretrained(model_name)
        
        if torch.cuda.is_available():
            print("Using GPU for model inference")
            self.device = "cuda"
            # GPU配置
            self.model = AutoModelForCausalLM.from_pretrained(
                model_name, 
                dtype=torch.float16,  # GPU上使用float16加速
                device_map="auto",  # 自动分配到GPU
                trust_remote_code=True
            )
        #...
```

### 经验总结 
#### 模型下载
- 直接运行代码下载容易断连下载失败，且默认下载到系统盘，空间可能不够
```
本地已有模型时的加载逻辑
Transformers 库的from_pretrained()方法遵循 **“本地优先，远程兜底”** 的原则，具体规则：
识别本地路径：
如果model_name是本地目录（如./qwen2-7b-instruct），直接加载本地模型文件；
如果model_name是 Hugging Face Hub 上的模型名（如Qwen/Qwen2-7B-Instruct），先检查本地缓存目录。
本地缓存目录位置：
默认缓存路径：~/.cache/huggingface/hub/（Linux/Mac）或C:\Users\<用户名>\.cache\huggingface\hub\（Windows）；
如果之前下载过该模型，缓存目录中会有模型文件，直接加载，不会重复下载。
```
- 使用wget下载速度较慢
- Qwen/Qwen2-7B-instruct可使用modelscope下载，速度较快
#### 其他
- 配环境的时候可以使用无卡模式运行，价格大幅下降
- 搜索引擎去除csdn `-site:csdn.net `