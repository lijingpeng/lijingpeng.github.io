title: How to customize Writable class in Hadoop
date: 2015-09-21 12:49:45
tags:
---
I'm trying to implement Writable class, but i have no idea on how to implement a writable class if in my class there is nested object, such as list, etc.
```java
public class StorageClass implements Writable{

public String xStr;
public String yStr;

public List<Field> sStor

//omitted ctors


@override
public void write(DataOutput out) throws IOException{
    out.writeChars(xStr);
    out.WriteChars(yStr);

    //WHAT SHOULD I DO FOR List<Field>

}

@override
public void readFields(DataInput in) throws IOException{
    xStr = in.readLine();
    yStr = in.readLine();

    //WHAT SHOULD I DO FOR List<Field>
}

}

public class SubStorage{
    public String x;
    public String y;
}

}
```
Following is the Field class:
```java
public final class Field implements Comparable<Field>, Serializable {

    private String name;
    private DataType dataType;
    private Object value;
    private FieldType fieldType;


    public Field(){

    }



    public  Field(String name, DataType dataType, FieldType fieldType){
        this(name, dataType, null, fieldType);
    }

    public  Field(String name, DataType type, Object value, FieldType fieldType){
        this.name = name;
        this.dataType = type;
        this.value = value;
        this.fieldType = fieldType;
    }
}





public enum FieldType {
    PRI, LOOKUP, SCD, VERSION, OTHER
}



public enum DataType {

    UNDEFINED(4) {
        public int getSizeInBytes(Object value) {
            return STRING.getSizeInBytes(value);
        }
    },

    STRING(4) {
        public int getSizeInBytes(Object value) {
            if (value == null) {
                return 0;
            }
            return super.getSizeInBytes(value) + (value.toString().length() * 2); // length + chars
        }
    },

    INT(4),
    LONG(8),
    DOUBLE(8),
    DATETIME(8),
    BOOLEAN(1),
    BYTE(1),
    FLOAT(4),
    SHORT(2),
    CHAR(2),
    DATE(8),
    TIME(8),

    BLOB(0) {
        public int getSizeInBytes(Object value) {
            if (value == null) {
                return 0;
            }
            return ((byte[])value).length;
        }
    };

    private final int sizeInBytes;

    private DataType(int sizeInBytes) {
        this.sizeInBytes = sizeInBytes;
    }

    public int getSizeInBytes(Object value) {
        return sizeInBytes;
    }
}
```
Serializing collections is quite simple.
```java
@Override
public void readFields(DataInput in) throws IOException {
    int size = in.readInt();
    list= new ArrayList<Field>(size);
    for(int i = 0; i < size; i++){
        Field f = new Field();
        f.readFields(in);
        list.add(f);
    }
}

@Override
public void write(DataOutput out) throws IOException {
    out.writeInt(list.size());
    for (Field l : list) {
        l.write(out);
    }
}
```