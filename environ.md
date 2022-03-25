create: 2022/3/25 22:36

fin: 2022/3/25 23:26

# environment var
v站冲浪有位朋友提到了环境变量。。大概看下
https://v2ex.com/t/842470#

显然的会想到shell里面的`export var=val`, `unset`等等。
```c
extern char **environ;
```
https://github.com/bminor/bash/blame/master/lib/sh/getenv.c#L38

```c
/* Add ENVSTR to the end of the exported environment, EXPORT_ENV. */
#define add_to_export_env(envstr,do_alloc) \
do \
  { \
    if (export_env_index >= (export_env_size - 1)) \
      { \
	export_env_size += 16; \
	export_env = strvec_resize (export_env, export_env_size); \
	environ = export_env; \
      } \
    export_env[export_env_index++] = (do_alloc) ? savestring (envstr) : envstr; \
    export_env[export_env_index] = (char *)NULL; \
  } while (0)
```

> s.  Bash now sets the extern variable `environ' to the export environment it
>     creates, so C library functions that call getenv() (and can't use the
>     shell-provided replacement) get current values of environment variables.


https://github.com/bminor/bash/blob/f3a35a2d601a55f337f8ca02a541f8c033682247/NEWS#L1276

这里environ都是extern的，去看下libc跟kernel，

glibc搜了一下。
```C
/* This must be initialized; we cannot have a weak alias into bss.  */
char **__environ = NULL;
weak_alias (__environ, environ)
```

https://elixir.bootlin.com/glibc/latest/source/posix/environ.c#L7


```c
int
unsetenv (const char *name)
{
  size_t len;
  char **ep;

  if (name == NULL || *name == '\0' || strchr (name, '=') != NULL)
    {
      __set_errno (EINVAL);
      return -1;
    }

  len = strlen (name);

  LOCK;

  ep = __environ;
  if (ep != NULL)
    while (*ep != NULL)
      {
	if (!strncmp (*ep, name, len) && (*ep)[len] == '=')
	  {
	    /* Found it.  Remove this pointer by moving later ones back.  */
	    char **dp = ep;

	    do
		dp[0] = dp[1];
	    while (*dp++);
	    /* Continue the loop in case NAME appears again.  */
	  }
	else
	  ++ep;
      }

  UNLOCK;

  return 0;
}


```

### 后记：
看bash的时候看到个这 姑且先记录一下，另C166里面有xmalloc的概念。嵌入式的,应该不是一个概念(bash里面这是safe ver of malloc/realloc)
https://www.keil.com/support/man/docs/c166/c166_xrealloc.htm

```c
// lib/sh/stringvec.c
char **
strvec_resize (array, nsize)
     char **array;
     int nsize;
{
  return ((char **)xrealloc (array, nsize * sizeof (char *)));
}

// braces.c
void *
xrealloc(p, n)
     void *p;
     size_t n;
{
  return (realloc (p, n));
}

// xmalloc.c
PTR_T
xrealloc (pointer, bytes)
     PTR_T pointer;
     size_t bytes;
{
  PTR_T temp;

#if defined (DEBUG)
  if (bytes == 0)
    internal_warning("xrealloc: size argument is 0");
#endif

  FINDBRK();
  temp = pointer ? realloc (pointer, bytes) : malloc (bytes);

  if (temp == 0)
    allocerr ("xrealloc", bytes);

  return (temp);
}
```

Ref:
https://www.gnu.org/software/libc/manual/html_mono/libc.html#Environment-Variables


TODO:
glibc工具链使用的环境变量 http://www.scratchbox.org/documentation/general/tutorials/glibcenv.html
环境变量存储位置（答题&用qemu模拟查下，main栈头部，即为libc的so位置？）https://stackoverflow.com/questions/532155/linux-where-are-environment-variables-stored
