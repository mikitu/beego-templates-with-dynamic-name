# A way to include templates/partials with dynamic name
This is an example of of how to include dynamic templates into other templates

## A. Parse all partial templates  on init
### 1. Load your partials
```go

var LoadedTemplates *template.Template

func init() {
    cwd, _ := os.Getwd()

    fpath := filepath.Join( cwd, beego.BConfig.WebConfig.ViewsPath, "path/to/partials" )
    var paths []string

    walk := func(path string, info os.FileInfo, err error) (error) {
        if err == nil && !info.IsDir() && filepath.Ext(path) == ".tpl" {
            paths = append(paths, path)
        }
        return err
    }
    err := filepath.Walk(fpath, walk)
    if (err != nil) {
        panic(err)
    }
    LoadedTemplates, _ = template.ParseFiles(paths...)
}
```
### 2. In your controller, register a template function
```go

func IncludeTemplate(name string, data interface{})(out interface{}) {
    base := path.Base(name)
    tpl := LoadedTemplates.Lookup(base)
    var buffer bytes.Buffer
    tpl.Execute(&buffer, data)
    return template.HTML(buffer.String())
}

func (ctrl *MyController) Prepare() {
	...
    beego.AddFuncMap("IncludeTemplate", IncludeTemplate)
}
```
### 3. In your template
```html
<div>
	{{$TplName := "tpl_name"}}
	{{template $TplName .}}
</div>

```
or

```html
<div>
	{{range .Widgets.Items}}
		{{IncludeTemplate .TemplateName .Data}}
	{{end}}
</div>

```
## b. Parse template on request
This method is similar but less optimized due to template.ParseFiles

TO DO: exmple
