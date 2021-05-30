{: .prompt}
In [1]:

```python
from PIL import Image
import os
```

{: .prompt}
In [2]:

```python
folder_to_scan = '/Users/ivanlengyel/datasets/mars_new_2019_w_borders/'
files = os.listdir(folder_to_scan)
saving_folder_name = './no_border/'
valid_file_types = ['.jpg']
```

{: .prompt}
In [3]:

```python
len(files)
```

{: .prompt}
Out [3]: 




    133



{: .prompt}
In [4]:

```python
valid_files = []
for file in files:
    f, fe = os.path.splitext(file)
    if fe in valid_file_types:
        valid_files.append(file)

full_path_valid_files = [folder_to_scan + file for file in valid_files]
for file_path in full_path_valid_files:

    file_name = os.path.basename(file_path)
    base_name, file_ext = os.path.splitext(file_name)
    im = Image.open(file_path)

    im_w = im.size[0]
    im_h = im.size[1]
    # delete superior border

    imc = im.crop((0,40,im_w,im_h)) #left bottom right upper

    f_name = saving_folder_name + base_name + file_ext
    imc.save(f_name)
    
```
