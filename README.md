# Articles
This repository provides a publishing gateway to our [website](https://supertype.ai) 

Any articles in Markdown format (`.md`), in the `/articles` subdirectory, will be published to the website's [Article](https://supertype.ai/notes) section.

### Slug
This automation considers the file name as the post slug i.e if the name of the file is `data-science-101.md` then the post slug is `/notes/data-science-101`.

### Folder
Alternatively, for every folder present in the repository, a post will be created. The content of the folder post will be taken from the `index.md` file present under the directory.

## Reference
For more details on using images, YAML headers, relative links etc, please refer to the plugin author's documentation: https://www.aakashweb.com/docs/git-it-write/writing-posts/

## Tips on Writing 
I often get asked how I write my articles. I put my thoughts together on [Writing better technical articles](articles/technicalwriting.md)