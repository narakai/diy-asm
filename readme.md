## vscode汇编

- [资料库](./index.html)

## demo
```
; .586
DATA SEGMENT; use16
        MESG DB 'hello masm','$'
DATA ENDS
CODE SEGMENT; USE16
    ASSUME CS:CODE,DS:DATA
BEG:    MOV AX,DATA
        MOV DS, AX
LAST:   MOV AH,9
        MOV DX, OFFSET MESG
        INT 21H
        MOV AH,4CH
        INT 21H;BACK TO DOS
        
CODE ENDS
END  BEG
```
## 资料
[vscode汇编插件masm-tasm](https://gitee.com/dosasm/masm-tasm)
[vscode汇编环境搭建](https://zhuanlan.zhihu.com/p/353500822)

## 步骤
1. [win批处理命令](https://www.cnblogs.com/gd-luojialin/p/12495226.html)
2. [dos环境命令](https://www.dosbox.com/wiki/Commands)
3. [dosbox命令行参数](https://www.dosbox.com/wiki/Usage)
