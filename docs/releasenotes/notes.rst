.. _releasenotes_notes:



4.5.0 (released Nov 30, 2016)
--------------------------

4.3.3 (released Dec 12, 2016)
-----------------------------
 
**Features**

- #2873: code-block overflow in latex (due to commas)
- #1060, #2056: sphinx.ext.intersphinx: broken links are generated if relative paths are used in intersphinx_mapping
- #2931: code-block directive with same :caption: causes warning of duplicate target. Now code-block and literalinclude does not define hyperlink target using its caption automatially.
- #2962: latex: missing label of longtable
- #2968: autodoc: show-inheritance option breaks docstrings

**Improvements**

- #2890: Quickstart should return an error consistently on all error conditions 
- #2870: flatten genindex columns’ heights.  
- #2856: Search on generated HTML site doesnt find some symbols  
- #2882: Fall back to a GET request on 403 status in linkcheck  
- #2902: jsdump.loads fails to load search index if keywords starts with underscore  
- #2900: Fix epub content.opf: add auto generated orphan files to spine.  
- #2899: Fix hasdoc() function in Jinja2 template. It can detect genindex, search collectly.  
- #2901: Fix epub result: skip creating links from image tags to original image files.  
- #2917: inline code is hyphenated on HTML.  
- #1462: autosummary warns for namedtuple with attribute with trailing underscore.  

**Bugs fixed**

- #2996: The wheel package of Sphinx got crash with ImportError

4.3.2 (released Dec 1, 2016)
-----------------------------