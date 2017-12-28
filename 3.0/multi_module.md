## Multi-module project


General project we recommand using the single module mode, it project is complicated, [multi-layer controller]((/doc/3.0/controller.html#toc-04e)) can be used to divided controoler by feature. If still it can't meet your requirements, try create multi-module project.

Use `--mode=module` to create a molti-module project.

```sh
thinkjs new demo --mode=module
```

### Project structure

There are differences between single-module and multi-module project in project structure:

* `src/common` to store common code
* `src/home` default module
* `src/xxx` modules added according to feature