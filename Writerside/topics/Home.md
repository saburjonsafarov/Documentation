# ParseViewById delegate

## Старый метод инициализации view {id="view_1"}

Старый метод был помечен как устаренный потому что при использовании делегата "lazy" новый 'view' не реагирует на
обращения. Причина заключаетса в правилах инициализации 'lazy' блока. "Lazy" инициализирует свой блок лишь в первое
обращение.

```
@Deprecated("Working is wrong, DO NOT USE!!")
@MainThread
inline fun <reified V : View> Fragment.parseViewById(@IdRes viewId: Int): Lazy<V> {
    return lazy(LazyThreadSafetyMode.NONE) { requireView().findViewById<V>(viewId) }
}
```

## Новая версия инициализации view:

В новом методе изменен блок инициализации  "view". Теперь блок инициализируется при обращении к данному полю. Решение
реализовано с созданием нового класса делегате.

```
class ParseViewByIdDelegate<T : View>(
    @IdRes private val id: Int,
    private val initEveryCallingTime: Boolean
) : ReadWriteProperty<Fragment, T> {

    private var view: T? = null
    override fun getValue(thisRef: Fragment, property: KProperty<*>): T {
        if (view == null || initEveryCallingTime) view = thisRef.view?.findViewById(id)
        return view ?: throw NullPointerException(this::class.java.name)
    }

    override fun setValue(thisRef: Fragment, property: KProperty<*>, value: T) {
        view = value
    }
}
```

Делегате функция:

```
fun <T : View> singleParseViewById(@IdRes id: Int, initEveryCallingTime: Boolean = true) =
    ParseViewByIdDelegate<T>(id, initEveryCallingTime)
```

Пример использования:

```
class MyFragment : Fragment(R.layout.fragment_my) {
 ...
    private val myProperty: RecyclerView by parseViewById(R.id.recyclerView)

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
       super.onViewCreated(view, savedInstanceState)
       myProperty.adapter = MyAdapter() //initialization before assignment.
    }
 ...
}
```

Author: Saburjon Safarov
<a href="https://t.me/SaburjonSafarov">telegram</a>


