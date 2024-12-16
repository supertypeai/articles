# Articles

This repository provides a publishing gateway to our [website](https://supertype.ai)

Any articles in Markdown format (`.md`), in the `/articles` subdirectory, will be published to the website's [Article](https://supertype.ai/notes) section.

## Guidelines

- Please don't use large images. Keep images to a reasonable size by sizing it down beforehand using Photoshop, GIMP, or any image editor of your choice
- Please keep images in the `_/images` directory, and link accordingly using relative linking (e.g. `![alt text](/_images/dbt.png)`) in markdown notation
- Use \*_less than 6 images_ per articles ideally
- Try to include linking to other, revelant articles on Supertype.ai's own website / blogs. This will help readers find related content and promote the work of other authors on the site.

### Slug

This automation considers the file name as the post slug i.e if the name of the file is `data-science-101.md` then the post slug is `/notes/data-science-101`.

### Taxonomy/Categories

You can have more than one category for a post. You should use them correctly. For the most part, this would be at least `knowledge` and `notes`. Very rarely would you use `internal-guides` for internal documentation that are OK for public consumption.

```yaml
taxonomy:
  category:
    - knowledge
    # - internal-guides
    - notes
```

### Folder

Alternatively, for every folder present in the repository, a post will be created. The content of the folder post will be taken from the `index.md` file present under the directory.

## Reference

For more details on using images, YAML headers, relative links etc, please refer to the plugin author's documentation: https://www.aakashweb.com/docs/git-it-write/writing-posts/

## Tips on Writing

I often get asked how I write my articles. I put my thoughts together on [Writing better technical articles](articles/technicalwriting.md)
