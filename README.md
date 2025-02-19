# Google Images Search

[![PyPI version](https://badge.fury.io/py/Google-Images-Search.svg)](https://badge.fury.io/py/Google-Images-Search)
[![Build Status](https://travis-ci.com/arrrlo/Google-Images-Search.svg?branch=master)](https://travis-ci.com/arrrlo/Google-Images-Search)
[![Codacy Badge](https://api.codacy.com/project/badge/Grade/b3d5259c67ca48a7bfe844b9721b6c19)](https://www.codacy.com/app/arrrlo/Google-Images-Search?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=arrrlo/Google-Images-Search&amp;utm_campaign=Badge_Grade)

![GitHub issues](https://img.shields.io/github/issues/arrrlo/Google-Images-Search.svg)
![GitHub closed issues](https://img.shields.io/github/issues-closed/arrrlo/Google-Images-Search.svg)
![GitHub closed pull requests](https://img.shields.io/github/issues-pr-closed/arrrlo/Google-Images-Search.svg)

![PyPI - Python Version](https://img.shields.io/pypi/pyversions/Google-Images-Search.svg)
![GitHub](https://img.shields.io/github/license/arrrlo/Google-Images-Search.svg?color=blue)
![GitHub last commit](https://img.shields.io/github/last-commit/arrrlo/Google-Images-Search.svg?color=blue)

## Installation

Before you continue you need to setup your Google developers account and project:  

[https://console.developers.google.com](https://console.developers.google.com)
(Among all of the Google APIs enable "Custom Search API" for your project)

[https://cse.google.com/cse/all](https://cse.google.com/cse/all)
(In the web form where you create/edit your custom search engine enable "Image search" option and and for "Sites to search" option select "Search the entire web but emphasize included sites")

After setting up you Google developers account and project you should have your developers API key and project CX

Install package from pypi.org:  

```bash
> pip install Google-Images-Search
```

## CLI usage

```bash
# without environment variables:

> gimages -k __your_dev_api_key__ -c __your_project_cx__ search -q puppies
```

```bash
# with environment variables:

> export GCS_DEVELOPER_KEY=__your_dev_api_key__
> export GCS_CX=__your_project_cx__
>
> gimages search -q puppies
```

```bash
# search only (no download and resize):

> gimages search -q puppies
```

```bash
# search and download only (no resize):

> gimages search -q puppies -d /path/on/your/drive/where/images/should/be/downloaded
```

```bash
# search, download and resize:

> gimages search -q puppies -d /path/ -w 500 -h 500
```

## Programmatic usage

```python
from google_images_search import GoogleImagesSearch

# if you don't enter api key and cx, the package will try to search
# them from environment variables GCS_DEVELOPER_KEY and GCS_CX
gis = GoogleImagesSearch('your_dev_api_key', 'your_project_cx')

# example: GoogleImagesSearch('ABcDeFGhiJKLmnopqweRty5asdfghGfdSaS4abC', '012345678987654321012:abcde_fghij')

#define search params:
_search_params = {
    'q': '...',
    'num': 1-50,
    'safe': 'high|medium|off',
    'fileType': 'jpg|gif|png',
    'imgType': 'clipart|face|lineart|news|photo',
    'imgSize': 'huge|icon|large|medium|small|xlarge|xxlarge',
    'imgDominantColor': 'black|blue|brown|gray|green|pink|purple|teal|white|yellow'
}

# this will only search for images:
gis.search(search_params=_search_params)

# this will search and download:
gis.search(search_params=_search_params, path_to_dir='/path/')

# this will search, download and resize:
gis.search(search_params=_search_params, path_to_dir='/path/', width=500, height=500)

# search first, then download and resize afterwards
gis.search(search_params=_search_params)
for image in gis.results():
    image.download('/path/')
    image.resize(500, 500)
```

## Inserting custom progressbar function

```python
from google_images_search import GoogleImagesSearch

def my_progressbar(url, progress):
    print(url + ' ' + progress + '%')

gis = GoogleImagesSearch(
    'your_dev_api_key', 'your_project_cx', progressbar_fn=my_progressbar
)

...
```

## Saving to a BytesIO object

```python
from google_images_search import GoogleImagesSearch
from io import BytesIO
from PIL import Image

# in this case we're using PIL to keep the BytesIO as an image object
# this way we don't have to wait for disk save / write times
# the image is simply kept in memory
# this example should display 3 pictures of puppies!

gis = GoogleImagesSearch('your_dev_api_key', 'your_project_cx')

my_bytes_io = BytesIO()

gis.search({'q': 'puppies', 'num': 3})
for image in gis.results():
    # here we tell the BytesIO object to go back to address 0
    my_bytes_io.seek(0)

    # take raw image data
    raw_image_data = image.get_raw_data()

    # this function writes the raw image data to the object
    image.copy_to(my_bytes_io, raw_image_data)

    # or without the raw data which will be automatically taken
    # inside the copy_to() method
    image.copy_to(my_bytes_io)

    # we go back to address 0 again so PIL can read it from start to finish
    my_bytes_io.seek(0)

    # create a temporary image object
    temp_img = Image.open(my_bytes_io)
    
    # show it in the default system photo viewer
    temp_img.show()
```
