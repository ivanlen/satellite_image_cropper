{: .prompt}
In [1]:

```python
from PIL import Image
import os
import itertools
```

{: .prompt}
In [2]:

```python
from PIL.ImageOps import flip, mirror
import PIL
```

{: .prompt}
In [3]:

```python
folder_to_scan = '/Users/ivanlengyel/datasets/mars_all_curated/'
files = os.listdir(folder_to_scan)
len(files)
```

{: .prompt}
Out [3]: 




    343



{: .prompt}
In [4]:

```python
valid_file_types = ['.jpg']
valid_files = []
for file in files:
    f, fe = os.path.splitext(file)
    if fe in valid_file_types:
        valid_files.append(file)
full_path_valid_files = [folder_to_scan+file for file in valid_files]
```

{: .prompt}
In [7]:

```python
def generate_1d_limits(wind, limit, thresh):
    x_left = []
    x_right = []
    if limit >= wind:
        x_lim_reached = False
        i = 0
        while not x_lim_reached:
            x_l = i * wind
            x_r = (i + 1) * wind

            if x_r <= limit:
                x_right.append(x_r)
                x_left.append(x_l)
            else:
                x_lim_reached = True
                # some extra padding
                if (x_r - limit) / wind < thresh:
                    x_r = limit
                    x_l = limit - wind
                    x_right.append(x_r)
                    x_left.append(x_l)
            i += 1
    return (x_left, x_right)


def generate_cropping_boxes_from_limits(x_left, x_rigth, x_bottom, x_top):
    croping_boxes = []
    x_lims = [(x_l, x_r) for x_l, x_r in zip(x_left, x_rigth)]
    y_lims = [(x_l, x_r) for x_l, x_r in zip(x_bottom, x_top)]
    bounding_boxes = list(itertools.product(x_lims, y_lims))
    for i in range(len(bounding_boxes)):
        ((x1, x2), (y1, y2)) = bounding_boxes[i]
        croping_boxes.append((x1, y1, x2, y2))
    return croping_boxes


def generate_cropping_boxes(image, cropping_window, tresh):
    image_width, image_height = image.size
    x_left, x_rigth = generate_1d_limits(cropping_window, image_width, tresh)
    x_bottom, x_top = generate_1d_limits(cropping_window, image_height, tresh)
    croping_boxes = generate_cropping_boxes_from_limits(x_left, x_rigth, x_bottom, x_top)
    return croping_boxes


def image_square_resize(im_input, new_size):
    im = im_input.copy()
    im = im.resize((new_size, new_size), PIL.Image.ANTIALIAS)    
    return im


def image_rotator(im_input, angle):
    if angle==90:
        return im_input.transpose(Image.ROTATE_90) 
    elif angle == 180:
        return im_input.transpose(Image.ROTATE_180) 
    elif angle == 270:
        return im_input.transpose(Image.ROTATE_270) 
    else:
        raise ValueError('angle not supported')    
        
        
def image_augmentator(im_input, return_orig = True):
    im_aug = []
    if return_orig:
        im_aug.append(im_input.copy())
    # 1.flip
    im_aug.append(flip(im_input.copy()))
    # 2. rot 180
    im_aug.append(image_rotator(im_input.copy(), 180))
    # 3. flip(rot_90)
    im_aug.append(flip(image_rotator(im_input.copy(), 90)))
    # 4. flip(rot_270)
    im_aug.append(flip(image_rotator(im_input.copy(), 270)))
    return im_aug        
```

{: .prompt}
In [8]:

```python
# class SizeCounter:
#     def __init__(self):
#         self.counter = {}
    
#     def add(self, size):
#         try:
#             self.counter[size]
#             self.counter[size] +=1
#         except KeyError:
#             self.counter[size] = 1
        
```

{: .prompt}
In [9]:

```python
# sizes = SizeCounter()
# for file in full_path_valid_files:
#     im = Image.open(file)
#     sizes.add(size=im.size)
    
# # sizes.counter
```

### No augmentation

{: .prompt}
In [17]:

```python
cropping_window = 512
saving_folder_name = './cropped_files_no_aug/'
padding_tresh = 0.25
resize = False
image_output_size = 512

os.makedirs(saving_folder_name, exist_ok=True)
```

{: .prompt}
In [18]:

```python
for file in full_path_valid_files:
    im = Image.open(file)
    croping_boxes = generate_cropping_boxes(im, cropping_window, padding_tresh) 
    file_name = os.path.basename(file)
    base_name, file_ext = os.path.splitext(file_name)
    for i,b in enumerate(croping_boxes):
        imc = im.crop(b) #left bottom right upper
        if resize: 
            imc = image_square_resize(imc,image_output_size )
        f_name = saving_folder_name + base_name +'_{}_'.format(i) + file_ext
        imc.save(f_name)
print(len(os.listdir(saving_folder_name)))
```

{: .prompt}
Out [18]: 

    4120


### Aumentation

{: .prompt}
In [19]:

```python
cropping_window = 512
saving_folder_name = './cropped_files_aug/'
padding_tresh = 0.25
resize = False
image_output_size = 512

os.makedirs(saving_folder_name, exist_ok=True)
```

{: .prompt}
In [20]:

```python
for file in full_path_valid_files:    
    im = Image.open(file)
    for ia, im in enumerate(image_augmentator(im)):
        croping_boxes = generate_cropping_boxes(im, cropping_window, padding_tresh) 
        file_name = os.path.basename(file)
        base_name, file_ext = os.path.splitext(file_name)
        for i,b in enumerate(croping_boxes):
            imc = im.crop(b) #left bottom right upper
            if resize: 
                imc = image_square_resize(imc,image_output_size )
            f_name = saving_folder_name + base_name +'_{}_{}_'.format(ia,i) + file_ext
            imc.save(f_name)
    
print(len(os.listdir(saving_folder_name)))
```

{: .prompt}
Out [20]: 

    20600


{: .prompt}
In [None]:

```python

```
