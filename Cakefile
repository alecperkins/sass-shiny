
fs      = require 'fs'
path    = require 'path'
marked  = require 'marked'
util    = require 'util'
rimraf  = require 'rimraf'

GA_SNIPPET = """
<script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','https://www.google-analytics.com/analytics.js','ga');

  ga('create', 'UA-9658094-5', 'auto');
  ga('send', 'pageview');

</script>
"""

STYLES = """
<style>
    .Site__ {
        max-width       : 760px;
        margin          : 0 auto;
        padding         : 1em;
    }

    .Nav {
        text-align      : right;
    }
    .Nav ._Link.-primary {
        float           : left;
    }

    h1, h2, h3, h4, h5, h6 {
        position        : relative;
    }

    .DeepLink {
        position        : absolute;
        right           : 100%;
        margin-right    : 0.5rem;
    }

    p img {
        vertical-align: middle;
    }

    p code {
        white-space: nowrap;
    }

    pre {
        overflow-x: scroll;
    }

    /* LAF */

    .Site__ {
        color           : #0c0c0c;
    }

    p {
        line-height     : 1.35;
    }

    p code {
        background      : #f0f0f0;
        border          : 1px solid #ddd;
        padding         : 0.1em;
    }

    .Nav {
        border-bottom   : 1px solid #ddd;
        padding-bottom  : 1em;
    }
    .Nav ._Link.-primary {
        font-weight     : 700;
    }
    .hljs {
        border          : 1px solid #ddd;
    }

    .DeepLink {
        color           : #ddd;
        text-decoration : none;
    }

    .LanguageTag {
        color           : #ddd;
    }

</style>
"""

DEEP_LINKING = """
<script>
    // Add deep links that nest based on heading hierarchy.
    (function(){
        var heading_counts = {};
        var heading_stack = []
        var heading_els = document.querySelectorAll('h1, h2, h3, h4, h5, h6');
        for(var i = 0; i < heading_els.length; i++) {
            var current_heading = heading_els[i];
            current_heading.level = parseInt(current_heading.tagName.substring(1));
            while(heading_stack[heading_stack.length - 1] && heading_stack[heading_stack.length - 1].level >= current_heading.level) {
                heading_stack.pop()
            }
            heading_stack.push(current_heading);
            var parent_id = '';
            if(heading_stack.length > 1) {
                parent_id = heading_stack[heading_stack.length - 2].id;
            }
            var text = current_heading.innerText;
            var slug = text.toLowerCase().replace(/[^\\w]+/g,'-').replace(/\\-+/g,'-').replace(/^\\-+/g,'').replace(/\\-+$/g,'');
            slug = parent_id + '/' + slug;
            if(!heading_counts[slug]) {
                heading_counts[slug] = 0;
            }
            heading_counts[slug]++;
            if(heading_counts[slug] > 1) {
                slug = slug + '-' + heading_counts[slug].toString()
            }
            current_heading.id = slug;
            current_heading.innerHTML = '<a class="DeepLink" href="#' + slug + '">#</a>' + current_heading.innerHTML
        }
    })();
</script>
"""

SYNTAX_HIGHLIGHTING = """
<link rel="stylesheet" type="text/css" href="//cdnjs.cloudflare.com/ajax/libs/highlight.js/8.3/styles/default.min.css" />
<script src="//cdnjs.cloudflare.com/ajax/libs/highlight.js/8.3/highlight.min.js"></script>
<script>
    (function(){

        // Add a label that indicates the language used by the block.
        var code_blocks = document.querySelectorAll('pre code');
        for(var i=0; i<code_blocks.length; i++) {
            var language = code_blocks[i].classList[0].replace('lang-','');
            var lang_tag = document.createElement('div');
            lang_tag.className = 'LanguageTag';
            lang_tag.innerText = language;
            code_blocks[i].parentElement.insertBefore(lang_tag, code_blocks[i]);
        }

        hljs.initHighlightingOnLoad();
    })();
</script>
"""

_wrapInBase = (title, content) -> """
    <!doctype html>
    <html>
        <head>
            <meta charset="utf-8" />
            <meta name="viewport" content="width=device-width">
            <title>#{ title }</title>
            #{ STYLES }
        </head>
        <body>
            <div class="Site__">
                <div class="_Header__">
                    <nav class="Nav">
                        <a class="_Link -primary" href="/">Shiny</a>
                        <a class="_Link" href="/specification/">Specification</a>
                        <a class="_Link" href="/helpers/">Helpers</a>
                    </nav>
                </div>
                <div class="_Content__">
                    #{ content }
                </div>
            </div>
            #{ SYNTAX_HIGHLIGHTING }
            #{ DEEP_LINKING }
            #{ GA_SNIPPET }
        </body>
    </html>
"""

_processFile = (source_file, output_folder, title) ->
    source_text = fs.readFileSync(source_file).toString()
    fs.mkdirSync(output_folder)
    output_filename = path.join(output_folder, 'index.html')
    output_text = marked(source_text)
    output_text = _wrapInBase(title, output_text)
    fs.writeFileSync(output_filename, output_text)
    util.log("\t#{ source_file } --> #{ output_filename }")

task 'build:pages', ->
    SOURCE_DIR  = path.join('.', 'docs')
    OUTPUT_DIR  = path.join('.', 'pages')

    if fs.existsSync(OUTPUT_DIR)
        util.log('Clearing old build...')
        rimraf.sync(OUTPUT_DIR)

    util.log 'Building pages...'
    _processFile(path.join('.', 'README.md'), OUTPUT_DIR, 'Shiny')
    _processFile(path.join('.', 'LICENSE'), path.join(OUTPUT_DIR, 'license'), 'License - Shiny')
    fs.readdirSync(SOURCE_DIR).forEach (filename) ->
        source_file = path.join(SOURCE_DIR, filename)
        output_folder = filename.split('.')
        output_folder.pop()
        title = output_folder.join('.')
        output_folder = path.join(OUTPUT_DIR, output_folder.join('.'))
        _processFile(source_file, output_folder, "#{ title[0].toUpperCase() }#{ title[1...] } - Shiny")

    util.log('Done.')


