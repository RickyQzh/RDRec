# Whats New? (NLP 25'Spring)
## 1. Instruction
### 1.1 How to use a Multi-GPU device (NLP lecture project)
#### (a) set device number when testing
        >> CUDA_VISIBLE_DEVICES=4 nohup python seq.py --data_dir ./data/{dataset}/ --cuda --batch_size 32 --checkpoint ./checkpoint/{dataset}/ > seq.log 2>&1 &
        >> CUDA_VISIBLE_DEVICES=5 nohup python topn.py --data_dir ./data/{dataset}/ --cuda --batch_size 32 --checkpoint ./checkpoint/{dataset}/ > topn.log 2>&1 &
        >> CUDA_VISIBLE_DEVICES=6 nohup python exp.py --data_dir ./data/{dataset}/ --cuda --batch_size 32 --checkpoint ./checkpoint/{dataset}/ > exp.log 2>&1 &
#### (b) Pretrain the RDRec-New
        >> nohup python pretrain.py --data_dir ./data/beauty/ --cuda --batch_size 64 --checkpoint ./checkpoint/beauty/ --ratio 1:1:1:3 > pretrain.log 2>&1 &

### 1.2. How to replace the llama2 model with other LLMs (RDRec-New)?
You can replace llama2 with other models in Stage1:Intersection Rationale Distillation(we use Qwen3-14B-AWQ) and you can get it easily by using our `./data_qwen_distillation/qwen_distillation.py`scripts.

You need to download a Qwen model(or other model) from a source like `ModelScope` or `Hugging Face` beforehand.Then replace `{your_path}` with your local model path,replace `{input_path}{output_path}` according to your local file.

```
	>> cd data_qwen_distillation
	>> python qwen_distillation.py \
		--model_path={your_model_path} \
		--input_json_path={input_path} \
		--output_json_path={output_path} \
		--max_batch_size=32 \
		-- save_every_n_batches=50
```
## 2. Method 
###  2.1. Short-range Attention Module with CNN

replace ```./model/module.py``` with  ```./model/module_CNN.py```

###  2.2. Qwen3：Improvement of the Base Model

replace ```./data``` with ```./data_qwen_distillation```

###  2.3. Two-stage Distillation

- replace ```./pretrain.py``` with ```./pretrain.py_2stage```
- replace ```./model/templates.py``` with ```./model/templates_2stage.py```
- replace ```./utils/utils.py``` with ```./utils/utils_2stage.py```
 
## 3. Results

(a) Replication Results (```./Results_0508_RDRec```)

(b) RDRec+CNN (```./Results_0514_RDRec+CNN_0.2```)

(c) RDRec+Qwen3-14B (```./Results_0522_RDRec+Qwen```)

(d) RDRec+CNN+Qwen3-14B (```./Results_0520_RDRec+CNN+Qwen_max```)
 
---

# RDRec (ACL'24)

## Paper - [[ArXiv]](https://arxiv.org/pdf/2405.10587) [[ACL Anthology]](https://aclanthology.org/2024.acl-short.6/)
- RDRec: Rationale Distillation for LLM-based Recommendation, **ACL 2024 Main (short)**.

## Note
- Please use the latest code released on **<u>June 11th, 2024</u>**.
- The checkpoints of the RDRec model for Step 2 were uploaded on [Google Drive](https://drive.google.com/drive/folders/1bwhliM4KN8pBdk5c0pRPDVCgTJbeOk0s) and [Baidu Drive](https://pan.baidu.com/s/15TQ6zi-ZHfPik02bjlPwRQ?pwd=eb3d ).
- The experimental setup follows [POD](https://github.com/lileipisces/POD). If there is any problem, please check our code or [[ArXiv]](https://arxiv.org/pdf/2405.10587).

## Instruction
### Step. 1 distill rationale before running RDRec

#### (a) Install llama 2 （download model weights and tokenizer）
        get the License from [the site](https://llama.meta.com/llama-downloads/)
        >> cd llama 
	    >> ./download.sh (License required)
        >> pip install -e .

#### (b) Test llama 2 environment  (under ./llama )
        >> torchrun --nproc_per_node 1 example_chat_completion.py \
          --ckpt_dir llama-2-7b-chat/ \
          --tokenizer_path tokenizer.model \
          --max_seq_len 512 --max_batch_size 6

#### (c) Rationale distillation  ({dataset}: beauty, sports, and toys.) (under ./RDRec )
        >> torchrun --nproc_per_node 1 data/{dataset}/distillation_{dataset}.py \
          --ckpt_dir llama/llama-2-7b-chat/ \
          --tokenizer_path llama/tokenizer.model \
          --max_seq_len 512 --max_batch_size 6

### Step. 2 train and test RDRec

#### (a) Install requirement (After installing torch)
        >> pip install transformers==4.36.2 -i https://pypi.python.org/simple
        >> pip install -r requirements.txt

#### (b) Pre-training ({dataset}: beauty, sports, and toys.) (under ./RDRec )
        >> python pretrain.py --data_dir ./data/{dataset}/ --cuda --batch_size 64 --checkpoint ./checkpoint/{dataset}/

#### (c) Recommendation inference 
        >> python seq.py --data_dir ./data/{dataset}/ --cuda --batch_size 32 --checkpoint ./checkpoint/{dataset}/
        >> python topn.py --data_dir ./data/{dataset}/ --cuda --batch_size 32 --checkpoint ./checkpoint/{dataset}/
        >> python exp.py --data_dir ./data/{dataset}/ --cuda --batch_size 32 --checkpoint ./checkpoint/{dataset}/


## Others
- All experiments, including rationale distillation, can be conducted on a **<u>single Nvidia GeForce RTX 3090 (24GB memory)</u>**. Reduce the batch size if you encounter an OOM error on some dataset.
- There are some fluctuations in RDRec's results for sequential recommendations. We reported average results in 10-trial runs in the paper  (See [t_test.py](https://github.com/WangXFng/RDRec/blob/main/utils/t_test.py) for more details). If the results are not ideal, please pre-train the model once again. 
- If you have any questions, please feel free to contact me at kaysenn@163.com.


## Citation
If this repository helps you, please cite:

	@article{wang2024rdrec,
	  title={RDRec: Rationale Distillation for LLM-based Recommendation},
	  author={Wang, Xinfeng and Cui, Jin and Suzuki, Yoshimi and Fukumoto, Fumiyo},
	  journal={arXiv preprint arXiv:2405.10587},
	  year={2024}
	}


## Acknowledge
- Code reference: [P5](https://github.com/jeykigung/P5), [POD](https://github.com/lileipisces/POD), and [llama 2](https://github.com/facebookresearch/llama).
- Thanks to Wei-Hsiang Huang's careful review, although RDRec independently generate user preferences and item attributes, the widely used review data (e.g., in [P5](https://github.com/jeykigung/P5) and [POD](https://github.com/lileipisces/POD)) could potentially leak information. The follow-up work will avoid this.
