Make file 

```make html```

or
```
sphinx-autobuild . _build/html 
# comment 'extensions.append('exhale')' in conf.py
```


Build file
Msg:
Generate from ros-doc-lite. Copied from generated to to _static, 

and change refer Link in html file. 

E.g. Labeled marker.html, search PointFloat32.html and change the href link as ./PointFloat32.html
