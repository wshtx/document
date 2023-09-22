# IDAE.md

## 配置方法注释模板

```text
变量param: groovyScript("def result=''; def params=\"${_1}\".replaceAll('[\\\\[|\\\\]|\\\\s]', '').split(',').toList(); for(i = 0; i < params.size(); i++) {if(i==0){result+='* @param ' + params[i] + ((i < params.size() - 1) ? '\\n' : '')}}; return result", methodParameters())

变量：groovyScript("return \"${_1}\" == 'void' ? null : '\\r\\n * @return ' + \"${_1}\"", methodReturnType())

abbreviation: *

template text:
*
 * @description: $end$
 * @author: Hetianxiang
 * @date: $DATE$
 $PARAM$ $RETURN$
 **/

```
