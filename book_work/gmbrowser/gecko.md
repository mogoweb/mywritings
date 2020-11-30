

```
alex@alex-MS-7C22:/work/gmbrowser/src/gecko/security/nss/cmd/addbuiltin$ make
/bin/sh: 1: ../../coreconf/nsinstall/Linux5.3_x86_cc_glibc_PTH_DBG.OBJ/nsinstall: not found
../../coreconf/rules.mk:392: recipe for target 'Linux5.3_x86_cc_glibc_PTH_DBG.OBJ/addbuiltin.o' failed
make: *** [Linux5.3_x86_cc_glibc_PTH_DBG.OBJ/addbuiltin.o] Error 127
```

修改 config/external/nss/Makefile.in

addbuiltin -n "sm2 ovssl certificates" -t C,C,C < ~/Downloads/sm2testovsslcn.der >> certdata.txt