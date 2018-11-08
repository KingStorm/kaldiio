# Kaldiio
[![Build Status](https://travis-ci.com/nttcslab-sp/kaldiio.svg?branch=master)](https://travis-ci.com/nttcslab-sp/kaldiio)

A pure python module for reading and writing kaldi ark files

## Dependencies

    numpy
    scipy
    six
    Python2.7, Python3.5, Python3.6

## Features
The followings are supported.

- Read/Write for archive formats: ark, scp
- Binary/Text - Float/Double Matrix: DM, FM
- Binary/Text - Float/Double Vector: DV, FV
- Compressed Matrix for loading: CM1, CM2, CM3
- Read/Write via a pipe: e.g. "ark: cat feats.ark |"
- Loading wav.scp

The followings are not **supported**

- Compressed Matrix for writing
- NNet2/NNet3 egs
- Lattice file


## Install 

```bash
pip install git+https://github.com/nttcslab-sp/kaldiio
```


## Usage
### Basic

```python
import kaldiio

d = kaldiio.load_ark('a.ark')  # d is a generator object
for key, array in d:
    ...

# === load_scp can load ark file as lazy dict
d = kaldiio.load_scp('a.scp')
for key in d:
    array = d[key]

# === Create ark file from numpy
kaldiio.save_ark('b.ark', {'key': array, 'key2': array2})
# Create ark with scp _file, too
kaldiio.save_ark('b.ark', {'key': array, 'key2': array2},
                 scp='b.scp')

# === load_ark and load_scp can accepts file descriptor, too
with open('a.ark') as fd:
    kaldiio.load_ark(fd)
with open('a.scp') as fd:
    kaldiio.load_scp(fd)

# === Writes arrays to sys.stdout
import sys
kaldiio.save_ark(sys.stdout, {'key': array})

# === Writes arrays for each keys
# generate a.ark
kaldiio.save_ark('a.ark', {'key': array})
# After here, a.ark is opened with 'a' (append) mode.
kaldiio.save_ark('a.ark', {'key2': array2}, append=True)
```

## Utility
### open_like_kaldi

``kaldiio.open_like_kaldi`` maybe a useful tool if you are familiar with Kaldi. This functions can performs as following,

```python
from kaldiio import open_like_kaldi
with open_like_kaldi('echo -n hello |', 'r') as f:
    assert f.read() == 'hello'
with open_like_kaldi('| cat > out.txt', 'w') as f:
    f.write('hello')
with open('out.txt', 'r') as f:
    assert f.read() == 'hello'

import sys
with open_like_kaldi('-', 'r') as f:
    assert f is sys.stdin
with open_like_kaldi('-', 'w') as f:
    assert f is sys.stdout
```

For example, there are gziped alignment file, how to open it:
```python
from kaldiio import open_like_kaldi, load_ark
with open_like_kaldi('gunzip -c exp/tri3_ali/ali.*.gz |', 'rb') as f:
    # Alignment format equals ark of IntVector
    g = load_ark(f)
    for k, array in g:
        ...
```

## parse_specifier

```python
from kaldiio import parse_specifier, open_like_kaldi, load_ark
rspecifier = 'ark:gunzip -c file.ark.gz |'
spec_dict = parse_specifier(rspecifier)
# spec_dict = {'ark': 'gunzip -c file.ark.gz |'}

with open_like_kaldi(spec_dict['ark'], 'rb') as fark:
    for key, array in load_ark(fark):
        ...
```

## Highlevel wrapper

### WriteHelper
```python
import numpy
from kaldiio import WriteHelper
with WriteHelper('ark,scp:file.ark,file.scp') as writer:
    for i in range(10):
        writer(str(i), numpy.random.randn(10, 10))
```

### ReadHelper
```python
from kaldiio import ReadHelper
for key, array in ReadHelper('ark: gunzip -c file.ark.gz |'):
    ...
```
