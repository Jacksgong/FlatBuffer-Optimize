# Flatbuffer Use Optimize
### 一. Tool Launguage
python.

### 二. Cause
1. __offset() method is not fast enough.
2. for get a varaible may need to calls __offset() method many times.

### 三. Effect
1. Add has set method for flatbuffer file.
2. Add member variables cache (include array variables).

### 四. Exp

origin:

	public int top() { int o = __offset(4); return o != 0 ? bb.getInt(o + bb_pos) : 0; }

convert to:

```
  public boolean has_top_cache = false;
  public int top_cache;
  public int top() { if ( has_top_cache ) { return top_cache; }  int o = __offset(4); top_cache = o != 0 ? bb.getInt(o + bb_pos) : 0;  has_top_cache= true; return top_cache; }
  public boolean hasSetValue_top = false; 
  public boolean hasSet_top() { if ( hasSetValue_top ) { return true; }  if ( has_top_cache  ) { return top_cache != 0; } int o = __offset(4); if (o == 0) { has_top_cache = true; top_cache = 0; return false; } else { hasSetValue_top = true; return true; } }
 ```
 
origin:

	public String fontFamily(int j) { int o = __offset(10); return o != 0 ? __string(__vector(o) + j * 4) : null; }
	
convert to:

```
  public int list_fontFamily_offset = -1;
  public android.util.SparseArray<String> fontFamily_cache = new android.util.SparseArray<String>();
  public String fontFamily(int j) { if ( fontFamily_cache.get(j) != null ) { return fontFamily_cache.get(j); }  int o = list_fontFamily_offset != -1? list_fontFamily_offset : __offset(10); String value =  o != 0 ? __string(__vector(o) + j * 4) : null;  list_fontFamily_offset = o; fontFamily_cache.put( j, value); return value; }
  public boolean hasSet_fontFamily() { if ( list_fontFamily_offset != -1 ) { return list_fontFamily_offset != 0; } list_fontFamily_offset = __offset(10); return list_fontFamily_offset!= 0; }
```

### 五. Tips
Flatbuffer is very fast from flatbuffer instream to avaliable object, but `__offset()` is very slow, so very frequently invoked almost not recommended. I tested 100,000 times `__offset()`, consuming greater than 50ms, but Java object directly accesse the same time only need 2~3ms. So such tool is come, but you need to pay attention to the increase of the GC.

