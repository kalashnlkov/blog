create: 2022/3/28 21:37

## 记录下碰到的魔法c

https://godbolt.org/z/GdfofM7dn

代码原型来自musl……

```c
#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <stdlib.h>
#include <limits.h>
#include <stdint.h>

#define ONES ((size_t)-1/UCHAR_MAX)
#define ALIGN (sizeof(size_t))
#define HIGHS (ONES * (UCHAR_MAX/2+1))
#define HASZERO(x) ((x)-ONES & ~(x) & HIGHS)

char *env[255] = {"A=B", "B=C", "HOME=/root"};
char **environ = env;

void __env_rm_add(char *old, char *new)
{
	static char **env_alloced;
	static size_t env_alloced_n;
	for (size_t i=0; i < env_alloced_n; i++)
		if (env_alloced[i] == old) {
			env_alloced[i] = new;
			free(old);
			return;
		} else if (!env_alloced[i] && new) {
			env_alloced[i] = new;
			new = 0;
		}
	if (!new) return;
	char **t = realloc(env_alloced, sizeof *t * (env_alloced_n+1));
	if (!t) return;
	(env_alloced = t)[env_alloced_n++] = new;
}
char *__strchrnul(const char *s, int c)
{
	c = (unsigned char)c;
	if (!c) return (char *)s + strlen(s);

#ifdef __GNUC__
	typedef size_t __attribute__((__may_alias__)) word;
	const word *w;
	for (; (uintptr_t)s % ALIGN; s++)
		if (!*s || *(unsigned char *)s == c) return (char *)s;
	size_t k = ONES * c;
	for (w = (void *)s; !HASZERO(*w) && !HASZERO(*w^k); w++);
	s = (void *)w;
#endif
	for (; *s && *(unsigned char *)s != c; s++);
	return (char *)s;
}

int unsetenv(const char *name)
{
	size_t l = __strchrnul(name, '=') - name;
	if (!l || name[l]) {
		errno = EINVAL;
		return -1;
	}
	if (environ) {
		char **e = environ, **eo = e;
		for (; *e; e++) {
            char _f ;
            _f = l[*e]; // LABEL
			// if (!strncmp(name, *e, l) && _f == '=')
            // // if (!strncmp(name, *e, l) && l[*e] == '=')
			// 	__env_rm_add(*e, 0);
			// else if (eo != e)
			// 	*eo++ = *e;
			// else
			// 	eo++;
        }
		if (eo != e) *eo = 0;
	}
	return 0;
}
void _test(const char* name)
{
    char _f;
    size_t l;
    l = name;
    char *it = name;
    //for (; it; it++) {
    //    printf("%c", it);
    //}
    printf("%d %s", l, l);

}
int main()
{
//    printf("%s\n%c", environ[0], environ[0][0]);
//    unsetenv("A");
    _test(environ[0]);
	return 0;

}
```

`_f = l[*e];` 这句汇编成了下面的没看懂。
```ASM
        mov     rax, QWORD PTR [rbp-8]
        mov     rdx, QWORD PTR [rax]
        mov     rax, QWORD PTR [rbp-16]
        add     rax, rdx
        movzx   eax, BYTE PTR [rax]
        mov     BYTE PTR [rbp-25], al
```