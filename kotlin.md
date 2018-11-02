+ ### by lazy 惰性加载

class FragmentUtil private constructor() {

    val homeFragment by lazy { SettingFragent() }

    companion object {
        val FragmentUtil by lazy { FragmentUtil() }
    }
    
}
