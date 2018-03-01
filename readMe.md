# 環境與硬體      
| item        | detail                             |
| :---------  | :--------                          |
| OS          | Windows 10 64 bit                  |  
| Anaconda    | Anaconda3-5.1.0-Windows-x86_64.exe |  
| Python      | Python 3.5                         |   
| perl        | strawberry-perl-5.24.3.1-64bit.msi |
| pyrouge     | pyrouge 0.1.3                      |
| ROUGE       | ROUGE-1.5.5                        |


# 安裝紀錄 
## 建立虛擬環境  
```bash
conda create -n tf121_py35 python=3.5  
```

## 切換虛擬環境  
```bash
activate tf121_py35
```
## 安裝pyrouge  
In the cmd.exe, run  
```
pip install pyrouge
```

## 下載ROUGE-1.5.5
Download ROUGE-1.5.5. You may download it from      
[https://github.com/andersjo/pyrouge/tree/master/tools/ROUGE-1.5.5]()  
我將ROUGE-1.5.5放在 C:\app\ROUGE-1.5.5，可自由選擇，資料夾名稱盡量避免使用中文、符號、空白

## 將ROUGE-1.5.5安裝位置指定給pyrouge   
請依自己ROUGE-1.5.5的安裝位置修改路徑
```bash
python C:\Anaconda3\envs\tf121_py35\Scripts\pyrouge_set_rouge_path  C:\app\ROUGE-1.5.5
```

## 測試pyrouge的運行  

建立測試程式 test.py
```
from pyrouge import Rouge155
r = Rouge155()
```
In the cmd.exe, You can run this python script , it should give no error
```python
python test.py
```

## 安裝Perl
You may download it from  
[http://strawberryperl.com]()  
本次下載perl版本為 5.24.3.1-64bit，下載網址：  
[http://strawberryperl.com/download/5.24.3.1/strawberry-perl-5.24.3.1-64bit.msi
strawberry-perl-5.24.3.1-64bit.msi]()  
有嘗試activestate版本的Perl但都失敗，沒有嘗試其他版本的StrawberryPerl，不確定其他Perl版本下，pyrouge使用中是否會出錯。  
安裝時若有使否要把Perl資料夾加到環境變數的選項，請打勾。  
接下來請確認是否有被加到系統變數(System variables)Path中，如沒有請自行將 C:\Strawberry\perl\bin 新增至環境變數。  
安裝玩Perl後，請重開機或重開cmd.exe，
## Perl安裝XML::DOM插件

```bash
ppm install XML::DOM
```

## 排除ROUGE-1.5.5的bug，如未排除會導致計算ROUGE時發生異常
手動刪除檔案 C:\app\ROUGE-1.5.5\data\WordNet-2.0.exc.db
```bash
c:
cd \app\ROUGE-1.5.5\data
perl WordNet-2.0-Exceptions/buildExeptionDB.pl ./WordNet-2.0-Exceptions ./smart_common_words.txt ./WordNet-2.0.exc.db
```

## 修改pyrouge\Rouge155.py的bug(如使用我fork的版本，而非使用pip install pyrouge安裝的人，跳過此步驟)  
我當下的路徑為(請依你的虛擬環境位置做更改)：  
C:\Anaconda3\envs\tf121_py35\Lib\site-packages\pyrouge\Rouge155.py  
找到def evaluate(self, system_id=1, rouge_args=None)，在我寫這份紀錄時的版本為319行，將command = [self._bin_path] + options 註解，新增command = ['perl'] + [self._bin_path] + options  
如下：
```python
    def evaluate(self, system_id=1, rouge_args=None):
        """
        Run ROUGE to evaluate the system summaries in system_dir against
        the model summaries in model_dir. The summaries are assumed to
        be in the one-sentence-per-line HTML format ROUGE understands.

            system_id:  Optional system ID which will be printed in
                        ROUGE's output.

        Returns: Rouge output as string.

        """
        self.write_config(system_id=system_id)
        options = self.__get_options(rouge_args)
        # command = [self._bin_path] + options
        command = ['perl'] + [self._bin_path] + options
        self.log.info(
            "Running ROUGE with command {}".format(" ".join(command)))
        rouge_output = check_output(command).decode("UTF-8")
        return rouge_output
```
如果沒有做這個修改會出現
```
OSError: [WinError 193] %1 不是有效的 Win32 應用程式。
```
或
```
OSError: [WinError 193] %1 is not a valid Win32 application.
```
### 測試pyrouge的安裝是否成功
Don't try to run 
```
python -m pyrouge.test
```
套件提供的測試語法有Bug，自行建立以下資料
```
◢ test_pyrouge
    ◢ model_summaries
        text.A.001.txt
    ◢ system_summaries
        text.001.txt
    rouge.py
```

rouge.py contains:
```python
from pyrouge import Rouge155
r = Rouge155()

r.system_dir = 'system_summaries'
r.model_dir = 'model_summaries'
r.system_filename_pattern = 'text.(\d+).txt'
r.model_filename_pattern = 'text.[A-Z].#ID#.txt'

output = r.convert_and_evaluate()
print(output)
output_dict = r.output_to_dict(output)
```

text.A.001.txt contains:
```
Dondon is cool, then run ROUGE
```
text.001.txt contains:
```
I only run ROUGE, then Dondon is cool
```

Output when running rouge.py:
```
2018-03-01 17:46:31,704 [MainThread  ] [INFO ]  Writing summaries.
2018-03-01 17:46:31,710 [MainThread  ] [INFO ]  Processing summaries. Saving system files to C:\Users\user\AppData\Local\Temp\tmp59kmbloh\system and model files to C:\Users\user\AppData\Local\Temp\tmp59kmbloh\model.
2018-03-01 17:46:31,710 [MainThread  ] [INFO ]  Processing files in system_summaries.
2018-03-01 17:46:31,711 [MainThread  ] [INFO ]  Processing text.001.txt.
2018-03-01 17:46:31,714 [MainThread  ] [INFO ]  Saved processed files to C:\Users\user\AppData\Local\Temp\tmp59kmbloh\system.
2018-03-01 17:46:31,714 [MainThread  ] [INFO ]  Processing files in model_summaries.
2018-03-01 17:46:31,715 [MainThread  ] [INFO ]  Processing text.A.001.txt.
2018-03-01 17:46:31,718 [MainThread  ] [INFO ]  Saved processed files to C:\Users\user\AppData\Local\Temp\tmp59kmbloh\model.
2018-03-01 17:46:31,721 [MainThread  ] [INFO ]  Written ROUGE configuration to C:\Users\user\AppData\Local\Temp\tmpbw1wif67\rouge_conf.xml
2018-03-01 17:46:31,721 [MainThread  ] [INFO ]  Running ROUGE with command perl C:\app\ROUGE-1.5.5\ROUGE-1.5.5.pl -e C:\app\ROUGE-1.5.5\data -c 95 -2 -1 -U -r 1000 -n 4 -w 1.2 -a -m C:\Users\user\AppData\Local\Temp\tmpbw1wif67\rouge_conf.xml
---------------------------------------------
1 ROUGE-1 Average_R: 1.00000 (95%-conf.int. 1.00000 - 1.00000)
1 ROUGE-1 Average_P: 0.75000 (95%-conf.int. 0.75000 - 0.75000)
1 ROUGE-1 Average_F: 0.85714 (95%-conf.int. 0.85714 - 0.85714)
---------------------------------------------
1 ROUGE-2 Average_R: 0.60000 (95%-conf.int. 0.60000 - 0.60000)
1 ROUGE-2 Average_P: 0.42857 (95%-conf.int. 0.42857 - 0.42857)
1 ROUGE-2 Average_F: 0.50000 (95%-conf.int. 0.50000 - 0.50000)
---------------------------------------------
1 ROUGE-3 Average_R: 0.25000 (95%-conf.int. 0.25000 - 0.25000)
1 ROUGE-3 Average_P: 0.16667 (95%-conf.int. 0.16667 - 0.16667)
1 ROUGE-3 Average_F: 0.20000 (95%-conf.int. 0.20000 - 0.20000)
---------------------------------------------
1 ROUGE-4 Average_R: 0.00000 (95%-conf.int. 0.00000 - 0.00000)
1 ROUGE-4 Average_P: 0.00000 (95%-conf.int. 0.00000 - 0.00000)
1 ROUGE-4 Average_F: 0.00000 (95%-conf.int. 0.00000 - 0.00000)
---------------------------------------------
1 ROUGE-L Average_R: 0.50000 (95%-conf.int. 0.50000 - 0.50000)
1 ROUGE-L Average_P: 0.37500 (95%-conf.int. 0.37500 - 0.37500)
1 ROUGE-L Average_F: 0.42857 (95%-conf.int. 0.42857 - 0.42857)
---------------------------------------------
1 ROUGE-W-1.2 Average_R: 0.34941 (95%-conf.int. 0.34941 - 0.34941)
1 ROUGE-W-1.2 Average_P: 0.37500 (95%-conf.int. 0.37500 - 0.37500)
1 ROUGE-W-1.2 Average_F: 0.36175 (95%-conf.int. 0.36175 - 0.36175)
---------------------------------------------
1 ROUGE-S* Average_R: 0.26667 (95%-conf.int. 0.26667 - 0.26667)
1 ROUGE-S* Average_P: 0.14286 (95%-conf.int. 0.14286 - 0.14286)
1 ROUGE-S* Average_F: 0.18605 (95%-conf.int. 0.18605 - 0.18605)
---------------------------------------------
1 ROUGE-SU* Average_R: 0.40000 (95%-conf.int. 0.40000 - 0.40000)
1 ROUGE-SU* Average_P: 0.22857 (95%-conf.int. 0.22857 - 0.22857)
1 ROUGE-SU* Average_F: 0.29091 (95%-conf.int. 0.29091 - 0.29091)
```
## 參考資料   
1. https://stackoverflow.com/questions/47045436/how-to-install-the-python-package-pyrouge-on-microsoft-windows/47045437#47045437  
1. http://blog.csdn.net/MerryCao/article/details/73477543  