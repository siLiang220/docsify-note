![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/utools_20221215212122.png)

```java
package org.example;

import java.lang.reflect.Field;
import java.util.*;

public class TransformEntity {

    private String TIME_KEY;

    private Integer RETURN_NUM;

    private Integer REROUTE_NUM;

    private Integer ABORT_NUM;

    private Integer IFSD_NUM;

    private Integer SILDEBACK_NUM;

    private Integer CHANGEPLANE_NUM;

    public String getTIME_KEY() {
        return TIME_KEY;
    }

    public void setTIME_KEY(String TIME_KEY) {
        this.TIME_KEY = TIME_KEY;
    }

    public Integer getRETURN_NUM() {
        return RETURN_NUM;
    }

    public void setRETURN_NUM(Integer RETURN_NUM) {
        this.RETURN_NUM = RETURN_NUM;
    }

    public Integer getREROUTE_NUM() {
        return REROUTE_NUM;
    }

    public void setREROUTE_NUM(Integer REROUTE_NUM) {
        this.REROUTE_NUM = REROUTE_NUM;
    }

    public Integer getABORT_NUM() {
        return ABORT_NUM;
    }

    public void setABORT_NUM(Integer ABORT_NUM) {
        this.ABORT_NUM = ABORT_NUM;
    }

    public Integer getIFSD_NUM() {
        return IFSD_NUM;
    }

    public void setIFSD_NUM(Integer IFSD_NUM) {
        this.IFSD_NUM = IFSD_NUM;
    }

    public Integer getSILDEBACK_NUM() {
        return SILDEBACK_NUM;
    }

    public void setSILDEBACK_NUM(Integer SILDEBACK_NUM) {
        this.SILDEBACK_NUM = SILDEBACK_NUM;
    }

    public Integer getCHANGEPLANE_NUM() {
        return CHANGEPLANE_NUM;
    }

    public void setCHANGEPLANE_NUM(Integer CHANGEPLANE_NUM) {
        this.CHANGEPLANE_NUM = CHANGEPLANE_NUM;
    }

    @Override
    public String toString() {
        return "Entity{" +
                "TIME_KEY='" + TIME_KEY + '\'' +
                ", RETURN_NUM=" + RETURN_NUM +
                ", REROUTE_NUM=" + REROUTE_NUM +
                ", ABORT_NUM=" + ABORT_NUM +
                ", IFSD_NUM=" + IFSD_NUM +
                ", SILDEBACK_NUM=" + SILDEBACK_NUM +
                ", CHANGEPLANE_NUM=" + CHANGEPLANE_NUM +
                '}';
    }

    public static void main(String[] args) throws Exception {
        List<TransformEntity> list = new ArrayList<>(3);
        TransformEntity transformEntity2019 = new TransformEntity();
        transformEntity2019.setTIME_KEY("2019");
        transformEntity2019.setRETURN_NUM(37);
        transformEntity2019.setREROUTE_NUM(13);
        list.add(transformEntity2019);

        TransformEntity transformEntity2020 = new TransformEntity();
        transformEntity2020.setTIME_KEY("2020");
        transformEntity2020.setRETURN_NUM(40);
        transformEntity2020.setREROUTE_NUM(14);
        list.add(transformEntity2020);

        TransformEntity transformEntity2021 = new TransformEntity();
        transformEntity2021.setTIME_KEY("2021");
        transformEntity2021.setRETURN_NUM(27);
        transformEntity2021.setREROUTE_NUM(11);
        list.add(transformEntity2021);

        List<List<Object>> convertedTable = convert(list, TransformEntity.class);
        for (List<Object> tempList : convertedTable) {
            for (Object string : tempList) {
                System.out.print(string + "  ");
            }
            System.out.println();
        }


    }

    private static List<List<Object>> convert(List<?> grandList, Class<?> clz)
            throws IllegalAccessException {
        Field[] declaredFields = clz.getDeclaredFields();
        List<List<Object>> convertedTable = new ArrayList<List<Object>>();

//多少个属性表示多少行，遍历行
        for (Field field : declaredFields) {
            field.setAccessible(true);
            ArrayList<Object> rowLine = new ArrayList<Object>();
            //list<T>多少个体类表示有多少列，遍历列
            for (int i = 0, size = grandList.size(); i < size; i++) {
                //if 判断如果是第一行将其设置为第一列
//                if(i == 0){
//                    rowLine.add(field.getName());
//                }
//                //新table从第二列开始，某一列的某个值对应旧table第一列的某个字段
//                else{
//                }
                Object obj = grandList.get(i);
                Object val = field.get(obj);
                rowLine.add(val);
            }
            convertedTable.add(rowLine);
        }
        return convertedTable;
    }
}
```