---                                                                                                                                         
layout: post                                                                                                                                
title: 'Extending Hatch With MkDocs'                                                                                            
date: '2023-12-09'                                                                                                                          
categories: opensource python                                                                                                                     
---   

## Context 

I recently have been using the project `hatch` for my Python project management. 
I love the tool so far. One command that I have been using a lot is the `hatch new` command. 
This command allows you to quickly create a Python project with all the essentials like a `README.md`, 
a `pyproject.toml`, linting, etc. 
I have been trying to be more consistent about writing documentation for my projects. 
One task I have been manually doing a lot is adding MkDocs to my Python projects. 
I thought it would be interesting to automatically add MkDocs to the `hatch new` templates. 

## Creating The Plugin 

I initialized a new project with `hatch new -i`. 

After completing the prompts, I have a `hatch-mkdocs-template` project. 

Hatch uses Pluggy as its plugin manager. It defines the following interface for Template plugins:


```python
# src/hatch/template/plugin/interface.py
class TemplateInterface:
    PLUGIN_NAME = ''
    PRIORITY = 100

    def __init__(self, plugin_config: dict, cache_dir, creation_time):
        self.plugin_config = plugin_config
        self.cache_dir = cache_dir
        self.creation_time = creation_time

    def initialize_config(self, config):
        """
        Allow modification of the configuration passed to every file for new projects
        before the list of files are determined.
        """

    def get_files(self, config):  # noqa: ARG002, PLR6301
        """Add to the list of files for new projects that are written to the file system."""
        return []

    def finalize_files(self, config, files):
        """Allow modification of files for new projects before they are written to the file system."""
```


I wrote the following `hooks.py`, `plugin.py`, and `file_templates.py`:

```python
# hooks.py
from hatchling.plugin import hookimpl

from .plugin import MkdocsTemplate

@hookimpl
def hatch_register_template():
    print("Registering!")
    return MkdocsTemplate
```

```python
# plugin.py
from hatch.template.plugin.interface import TemplateInterface
from hatch.template import find_template_files
from . import file_templates
class MkdocsTemplate(TemplateInterface):
    PLUGIN_NAME = 'mkdocs-template'

    def get_files(self, config):
        return list(find_template_files(file_templates))
```

```python
# file_templates.py
from hatch.template import File
from hatch.utils.fs import Path

class MkdocsYaml(File):
    TEMPLATE = """\
site_name: {project_name} Docs
nav:
    - Home: 'index.md'
theme: {theme}
"""
    def __init__(self, template_config: dict, plugin_config: dict):
        # defaults 
        theme = "readthedocs"

        if "theme" in plugin_config:
            theme = plugin_config["theme"]
        super().__init__(Path("mkdocs.yaml"), self.TEMPLATE.format(theme=theme,**template_config))

class IndexMd(File):
    TEMPLATE = """\
# {project_name}
{proj_description}

## Features 
- Awesome Feature!
"""
    def __init__(self, template_config: dict, plugin_config: dict):
        #defaults 
        proj_description = "This is a super cool project."

        if template_config["description"] != "":
            proj_description = template_config["description"]
        super().__init__(Path("docs/index.md"), self.TEMPLATE.format(proj_description=proj_description,**template_config))
```

I registered the plugin in the `pyproject.toml`: 


```toml
[project.entry-points.hatch]
mkdocs = "hatch_mkdocs_template.hooks"
```

That is pretty much the whole plugin. For more information about creating plugins, look at 
[these](https://hatch.pypa.io/latest/plugins/about/)  `hatch` docs. 


## Thoughts After Creating The Plugin
* I really like the `File` abstraction that `hatch` uses for templates. 
I previously wrote a similar abstraction in a project, 
and I chose to make all templates use Jinja2. However, forcing the user to use Jinja2 templates feels like the wrong choice now. 
It is much more flexible to take a string and allow the user to use whatever templates they want. 


```python
# src/hatch/template/__init__.py
class File:
    def __init__(self, path: Path | None, contents: str = ''):
        self.path = path
        self.contents = contents
        self.feature = None

    def write(self, root):
        if self.path is None:  # no cov
            return

        path = root / self.path
        path.ensure_parent_dir_exists()
        path.write_text(self.contents, encoding='utf-8')
```

* One thing I disliked is that in `TemplateInterface`, `template_config` and `plugin_config` variables are dictionaries. 
I found myself glancing at the code to understand what variables they contained. 
I feel like I would appreciate them being a data class or something. 
However, I do realize the dict allows for lots of flexibility, so I feel conflicted. 

* The Template plugin is not documented in the `hatch` docs currently. 
After glancing at the discussions, I learned that the developer is not satisfied with the API yet, so they haven’t documented it. 
They felt like it was not very scalable yet. 
I look forward to seeing how the interface changes and writing more template plugins in the future. 

You can track the discussion [here](https://github.com/pypa/hatch/discussions/143#discussioncomment-7796761).

## Closing Thoughts

It was really easy to write this plugin. 
I love the `hatch` project. 
If you haven’t checked out the code base, you definitely should. 
The maintainer is doing a great job. 
This project is the first example of Python plugin architecture I have been exposed to and extending it was great. 
I’m looking forward to seeing how the project grows.

My example plugin can be found [here](https://github.com/rod7760/hatch-mkdocs-template/tree/master).
