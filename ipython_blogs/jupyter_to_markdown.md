# Understand *.ipynb* file

We use jupyter notebook for writng code and report in single file. We can compile the notebook to outcome beautiful HTML, markdown or even slides. But have you ever wondered how this all is done? Have ever felt to modify the outcome of HTML or markdown from jupyter notebook? Don't worry we will learn this all in this tutorial. We will see what jupyter notebook is made of. And how we can selectively modify the outcome of the conversion of jupter notebook to any other format.

So we will first understand what jupyter notebook is made of. We will read the *.ipynb* file from python and then understand its content. For this we will make one notebook file in which we will have code, markdown content, plot and some printed outcomes. This convers most of the usage of the jupyter notebook file. We will not conver how widgets and other things are output from jupyter notebook for now being.

[This is the notebook]{https://github.com/ddrathod121294/blogging01/blob/591def9953c75349ea5a76d682af4cce4f1ea74a/ipython_blogs/notebook1.ipynb)

```py
filepath = '.\notebook1.ipynb'
```

Jupyter notebook arranges the data in JSON format. So we will import the JSON and parse the data of the file.


```python
import json
with open(filepath,'r') as f:
    str1 = f.read()
nb1 = json.loads(str1)
```

here ```nb1``` is the dict like object, with key and value pairs. The content of the notebook is in ```nb1['cells']```. We will deeply look into this soon. ```nb1['metadata']``` contains the metadata of the jupyter notebook. These metadata can be defined in the notebook from ```Edit > Edit Notebook Metadata ``` menubar. Metadata contains the information about the type of language used and other things. We can modify thie metadata of the file. But it is not advisable (unless you know what you are doing) and required right now. This metadata is mainly used for formating and other information. This is not printed or outcome anywhere. ```nb1['nbformat']``` contains the format of the jupyter notebook. I think of it as a version of the notebook. Currently, this is 4. Similarly, ```nb1['nbformat']``` value is 5.

We will mainly work with ```nb1['cells']``` in this tutorial.


```python
print(list(nb1.keys()))
```

    ['cells', 'metadata', 'nbformat', 'nbformat_minor']
    

### Understanding cells and its content

```nb1['cells']``` is the list. Each element in the list is dict like object. The keys of the dict varies according to the type of the cell, as shown below. We will go through each key step by step.


```python
cells = nb1['cells']
n = 0
print(f"for content type = {cells[n]['cell_type']}")
print(list((cells[n].keys())))
n = 3
print(f"for content type = {cells[n]['cell_type']}")
print(list((cells[n].keys())))
```

    for content type = markdown
    ['cell_type', 'id', 'metadata', 'source']
    for content type = code
    ['cell_type', 'execution_count', 'id', 'metadata', 'outputs', 'source']
    

```cell_type``` contains the information about the tye of the cell, whether code or markdown type of the cell. ```id``` is the unique id of each cell. ```metadata``` contain the meta information about the cell. This serves the same purpose as the metadata of the file, as explained above. We will use this meta information mainly to know whether to omit the cell or not for outcome. ```metadata``` contains the dict like object with key-value pair. It contains ```tags``` for each cell. We can give tags to certain cell to omit that particular cell in the outcome.

You can go to ```View > Cell Toolbar > Tags``` to view the tags. We can then add the tags to the cell. We will use following tags as per the requirements:-
- remove-input :- to remove the input of the cell
- remove-output :-  to remove the output of the cell. This will be used for code type cell mainly.
- remove-cell :- to remove the cell itself. It removes both the input and output.

```source``` contains the content of the cell. If it is markdown type of cell then it contains the text or if it is code type of cell then it contains the code inside the cell.

Now for code type of cell there are some extra key-value pair to account for output and execution number of the cell. By now you would have guessed what is what, but for the sake of bravity...

```execution_count``` contains the count of the run. When you run the cell the count increases by one. If you run the same cell 5 times then you will see its count to be 5.

```outputs``` contains the output of the code. It contains the list of all the outputs. It can be list plot, image or some text to be printed or nothing.

### Understanding code cell output
Before putting all things togather we need to understand how the ```outputs``` varies with different types of data and how we can compile it togather. Do note that, only the code cell contains the output. Here 4th cell is the code cell and it outputs plot and prints one statement. So the ```outputs``` cell contain list of 2 dict like elements. The first dict is for the image of the plot in *.png* format. The second dict is for the text. We can get this information from the ```output_type``` key of the element, as shown below.


```python
output = cells[3]['outputs']
print(output[0]['output_type'])
print(output[1]['output_type'])
```

    display_data
    stream
    

To access the data of the respective items we need to use different type of keys. Like for image type of data, the image is in ```data``` keys of that element. and for ```stream``` i.e. text like object, the data is in ```text``` key. So we will accordingly access the data of the output depending upon its type.

Now we will first try to put all the inputs and outputs togather and see the result.

## Preparing first *.md* file

We will concatenate all the ```source``` i.e. inputs of all the cells. But if the cell is code type then we have to append its output before moving onto the next cell. So we will have to write nested kind of if;else to tackle this situation.

```source``` contains the input text of the cell. This is the list of all the lines. So we will join the list and make it single string before appending it to final string. If the cell type is code, then we will have to enclose the code in three ``` ` ``` (Backticks). We will also append `\n` at the end of each cell because in markdown it does not matter how many linebreaks you give, if it is more than one then it only consideres one.


```python
# creating the blank string wherein we will add all the text.
md_output = ''
# iterating over all cells
for cell in cells:
    str1 = ''.join(cell['source']) + '\n'
    if cell['cell_type']=='markdown':
        md_output += str1
    
    if cell['cell_type'] == 'code':
        md_output += "\n```py\n" + str1 + "\n```\n"
```

Here we did not append any code output. But without it the markdown file does not make uch sense. So we will see how to do it now.

### Appending code output

If the code output (inside ```outputs``` key of the cells) is image type then we will store this image in our local drive and the give the reference to it in the markdown in markdown syntex. If the code output is text, then we can directly append it to the file. So we will modify the above code a bit to accomodate this facility.


```python
# creating the blank string wherein we will add all the text.
md_output = ''

# we will create one counter for creating different names for images.
img_num = 0
# we will store images in the dict with key as its name.
images = {}
# iterating over all cells
for cell in cells:
    str1 = ''.join(cell['source']) + '\n'
    if cell['cell_type']=='markdown':
        md_output += str1
    
    if cell['cell_type'] == 'code':
        md_output += "\n```py\n" + str1 + "```\n"
        
        outputs = cell['outputs']
        # we will iterate over all outputs and append it consecutively.
        for output in outputs:
            if output['output_type'] == 'display_data':
                # getting image byte data
                img = output['data']['image/png']
                img_name = 'output_png_' + str(img_num) + '.png'
                img_num += 1
                
                images[img_name] = img
                str1 = "![png](https://github.com/ddrathod121294/blogging01/blob/base/ipython_blogs/images/" + img_name + "?raw=true)"
                md_output += '\n' + str1 + '\n'
            if output['output_type']=='stream':
                str1 = "```py\n" + ''.join(output['text']) + "```"
                md_output += '\n' + str1 + '\n'
```

We are done with compiling the *.ipynb* file. We have to now store the *.md* file and images in the local drive.

### Saving the markdown and images.

We will write the ```md_output``` string to the *.md* file and write the images to *.png* file. We will store the imges in the same folder as in *.md* file, or else we will have to change path of the image in markdown text.

Before we store the image we will have to decode it to byte type with base-64.


```python
import base64

filename = 'file1.md'
with open(filename,'w') as f:
    f.write(md_output)

# we will iterate over all the images and save.
# do note that the .png is byte like file

for img_name in images:
    with open(img_name,'wb') as f:
        img = images[img_name]
        img1 = base64.b64decode(img)
        f.write(img1)
```

Done, We have successfully converted the *.ipynb* file to *.md* markdown file. You can open the file with markdown supported software like VB-code and look at the output. Similarly we can code and convert the file to HTML or any other format, if we know the syntex of the output file.

I hope this tutorial helps.
