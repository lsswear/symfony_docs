## 4.3.5发布

[发布修改内容](https://symfony.com/blog/symfony-4-3-5-released)

根据英文文档的发布内容看，与3.4版本相比无主要修改内容，都是bug修改。

## 3.4、4.3、4.4维护内容

https://symfony.com/blog/a-week-of-symfony-667-7-13-october-2019

### 4.4主要修改

+ String:核心类重命名为Byte/CodePoint/UnicodeString
+ Console:用跨平台并永远可用stream_isatty取代posix_isatty
+ String:join()方法添加参数$lastGlue

```
public function join(array $strings, string $lastGlue = null): parent
    {
        $str = clone $this;
        //$str->string = implode($str->string, $strings);
        $tail = null !== $lastGlue && 1 < \count($strings) ? $lastGlue.array_pop($strings) : '';
        $str->string = implode($this->string, $strings).$tail;
        return $str;
    }
```
