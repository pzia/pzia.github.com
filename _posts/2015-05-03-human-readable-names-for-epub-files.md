---
layout: post
title: "Human readable names for epub files"
description: "Metadatas from epub"
category: datas
tags: ["keepmydatas", "epub"]
---
{% include JB/setup %}

[Kobo](https://store.kobobooks.com/fr-fr/) builds nice eReaders.

Their store allow you to download (allas one by one) your epub files (more later on the DRM case).
They come with a hashed name, and if you want to keep a human readable library, this is not convenient (at all).

As part of my pet project [KeepMyDatas](https://github.com/pzia/keepmydatas), I wanted to rename to something more friendly and put them into a tree made of "Author"/"Title".

- First experiment with [Ebooklib](https://github.com/aerkalov/ebooklib) was interesting but non conclusive because ebooklib fails to read epub without a proper *identifier* metadata.
- Unzipping and reading by hand was more sucessful.

        ns = {
            'n':'urn:oasis:names:tc:opendocument:xmlns:container',
            'pkg':'http://www.idpf.org/2007/opf',
            'dc':'http://purl.org/dc/elements/1.1/'
        }

        # Read from the file
        zf = zipfile.ZipFile(fname)

        # find the contents metafile
        txt = zf.read('META-INF/container.xml')
        tree = etree.fromstring(txt)
        cfname = tree.xpath('n:rootfiles/n:rootfile/@full-path',namespaces=ns)[0]

        # grab content
        cf = zf.read(cfname)
        tree = etree.fromstring(cf)
        px = tree.xpath('/pkg:package/pkg:metadata',namespaces=ns)
        p = px[0]

        # remap datas
        res = {}
        for s in ['title','language','creator','date','identifier']:
            try :
                r = p.xpath('dc:%s/text()'%(s),namespaces=ns)
                v = r[0]
            except :  
                v = "Unknown"
            res[s] = v.strip()
        return res

An then you may rename your files with proper names.

Files are [KmdEpub.py](https://github.com/pzia/keepmydatas/blob/master/src/KmdEpub.py) and [epub_rename.py](https://github.com/pzia/keepmydatas/blob/master/src/epub_rename.py)
