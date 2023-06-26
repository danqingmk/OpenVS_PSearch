# What is PSearch_VS ?
 Pharmacophore modeling tool use PSearch(https://github.com/meddwl/psearch).
 Batch pharmacophore screening use script `pharm_screen`.



# How to use this tool ?

### 1 Bulid the environment

1. pip install psearch             
2. pip install pmapper == 0.3.1
   
### 2 Us Exmample 

* 1 prepare_datatset -i $PROJECT_DIR/input.smi -c 4
* 2 psearch -p $PROJECT_DIR -c 4
* 3 python pharm_screen.py --screen_path `` --prepare_db '' --screen_db '' --file_label '' --pharms '' --cpus ''
