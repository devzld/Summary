+ ### by lazy 惰性加载，线程安全

class FragmentUtil private constructor() {

    val homeFragment by lazy { SettingFragent() }

    companion object {
        val FragmentUtil by lazy { FragmentUtil() }
    }
    
}
