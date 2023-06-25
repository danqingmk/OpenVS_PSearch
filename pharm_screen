import os
import pandas as pd
import argparse
import multiprocessing as mp
import sys
def get_sdf(ref_sdf,file,pharm,index):
    if not os.path.exists(os.path.join(sdf_path,pharm)):
        os.makedirs(os.path.join(sdf_path,pharm))
    out_sdf = os.path.join(sdf_path,pharm,file.split('_')[0]+'.sdf')
    file_info = open(ref_sdf, 'r').read().split('$$$$\n')
    for j,row in enumerate(file_info):
        if j == int(index):
            f = open(out_sdf, 'a+')
            f.writelines(row+'$$$$\n')
            f.close()
def split_file(file):
    if len(os.listdir(smi_path))!=0:
        for i in os.listdir(smi_path):
            if i == 'data':
                continue
            os.remove(os.path.join(smi_path,i))
    if len(os.listdir(db_path))!=0:
        for i in os.listdir(db_path):
            os.remove(os.path.join(db_path,i))
    symbol_num = file.split('_')[-4]
    count = 0
    for j,line in enumerate(open(os.path.join(path,file)).readlines()):
        if j == 0:
            continue
        file_num = count//5000
        out_file = os.path.join(smi_path,str(symbol_num)+'_'+str(file_num)+'.smi')
        f_o = open(out_file,'a+')
        smi = line.strip().split(',')[0]
        f_o.write(smi+'\n')
        f_o.close()
        count+=1
def comrun(smi,file):
    if smi =='smi':
        str = 'python {1} ' \
              '-i {2}/temp_smi/{0}.smi ' \
              '-o {2}/temp_db/{0}.db -u 4 -n 10 '.format(file.split('.smi')[0],prepare_db,working_path)
        os.system(str)
    if smi == 'pharm':
        for pharm in pharms:
            str1 = 'python {1} ' \
                  '-q {2}/{3}.pma ' \
                  '-d {2}/temp_db/{0}.db ' \
                  '-o {2}/pharm_screen/{3}/aurorab_{0}.txt ' \
                  '-c 2'.format(file.split('.smi')[0],screen_db,working_path,pharm)
            os.system(str1)

def get_index(file):
    for pharm in pharms:
        if not os.path.exists(os.path.join(pharm_screen, pharm)):
            os.makedirs(os.path.join(pharm_screen, pharm))
        pharm1= '{}/pharm_screen/{}/'.format(working_path,pharm)
        pharm_file = pharm1+'aurorab_{}.txt'.format(file.split('.')[0])
        if os.path.exists(pharm_file):
            t = 0
            for line in open(pharm_file,'r').readlines():
                t+=1
                smiles= line.strip().split('	')[0]
                index = line.strip().split('	')[1]
                ref_sdf = os.path.join(db_path,file.split('.')[0]+'_conf.sdf')
                get_sdf(ref_sdf,file,pharm,index)


def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument('--screen_path', required=True, help='we must give this para')
    parser.add_argument('--working_path', required=True, help='we must give this para')
    parser.add_argument('--prepare_db', required=True, help='we must give this para')
    parser.add_argument('--screen_db', required=True, help='we must give this para')
    parser.add_argument('--file_label', required=True, help='we must give this para')
    parser.add_argument('--pharms', default=['tr13_pharm5_5','tr6_pharm5_2'], nargs='*')
    parser.add_argument('--cpus', default=0, type=int)
    parser.add_argument('--states', required=True, type=int)
    parser.add_argument('--start', default=1, type=int)

    args = parser.parse_args()
    return args

## first ,we need to move the dockvina to sdf file path
## second ,the computer should have obabel and autodock & vina
args = parse_args()
cpus = args.cpus
path = args.screen_path
working_path = args.working_path
prepare_db =args.prepare_db
screen_db =args.screen_db
file_label = args.file_label
states = args.states
start =args.start
pharms = args.pharms
# path = '/data/jianping/bokey/OCAICM/dataset/aurorab/bigscale/26-1/'  #need sceen machine learning file dir
# working_path = '/data/jianping/bokey/OCAICM/dataset/aurorab'   # sceen pharm generate file dir
# prepare_db ='/data/jianping/bokey/OCAICM/dataset/psearch-master/psearch/prepare_db.py'
# screen_db = '/data/jianping/bokey/OCAICM/dataset/psearch-master/psearch/scripts/screen_db.py'
# file_label = 'Enamine_REAL_HAC_26_Part_1_CXSMILES_{}w_screen_0.8_DNN.csv' #file sample
# states = 201
# start = 46
# pharms = []

count = 0#38406654
num=0
smi_path = os.path.join(working_path,'temp_smi')
db_path = os.path.join(working_path,'temp_db')
sdf_path = os.path.join(working_path,'save_sdf')
pharm_screen = os.path.join(working_path,'pharm_screen')

if not os.path.exists(smi_path):
    os.mkdir(smi_path)
if not os.path.exists(db_path):
    os.mkdir(db_path)
if not os.path.exists(sdf_path):
    os.mkdir(sdf_path)
    if not os.path.exists(pharm_screen):
        os.mkdir(pharm_screen)


for state in range(states):
    state = (state+start)*100
    file = file_label.format(state)
    file_path = os.path.join(path,file)
    if not os.path.exists(file_path):
        continue
    for line in open(file_path,'r').readlines():
        if line.startswith('cano'):
            continue
        smiles = line.split(',')[0].split('\t')[0].split(' ')[0]
        subfile = os.path.join(smi_path,file.split('_')[-4]+'_'+str(num)+'.smi')
        f= open(subfile,'a+')
        f.write(smiles+'\n')
        f.close()
        count+=1
        if count%10000==0 and count!=0:
            num+=1
    file_num = len(os.listdir(smi_path)) if cpus ==0 else cpus
    split_file(file)
    p = mp.Pool(processes=int(file_num))
    for file in os.listdir(smi_path):
        p.apply_async(comrun, args=('smi',file))
    p.close()
    p.join()

    p = mp.Pool(processes=int(file_num))
    for file in os.listdir(smi_path):
        p.apply_async(comrun, args=('pharm', file))
    p.close()
    p.join()

    p = mp.Pool(processes=int(file_num))
    for file in os.listdir(smi_path):
        p.apply_async(get_index, args=(file,))
    p.close()
    p.join()





