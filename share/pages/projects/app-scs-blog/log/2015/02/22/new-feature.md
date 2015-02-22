title: New Feature
created: 2015-02-22 03:34:54

# Sub Directory Listings

So I have now added a new function, which allows for the usage of sub
directories as the blog listings! this means that you can have a blog archive
in one location and then put the files in another location which may not be
directly below the current blog page!

## Usage

Modify the plugin header for the blog archive page to something similar to the
following:

```
<meta name="plugins" content="BlogArchive { max_depth: 4, min_depth: 1, sub_dir: log }" />
```

Assuming your filename is `archive.html` (or `.md`), then this will look for
the files in `archive/log` and sub folders, instead of just in the `archive`
folder. This is useful when you want to have the archive at
`example.com/archive` and the actual blog posts at `example.com/blog/post/`, as
you can change up directories.

Just remember that the plugin starts looking (in the case of the example
anyway) in the `archive` folder, so if you are trying to get to something on
the same directory level, your plugin config will look like this:

```
<meta name="plugins" content="BlogArchive { max_depth: 4, min_depth: 1, sub_dir: ../blog }" />
```
