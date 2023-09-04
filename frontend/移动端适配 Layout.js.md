```javascript
import { Platform, Dimensions, StyleSheet } from "react-native";  
  
import { getStatusBarHeight, getBottomSpace } from "../Util/IPhoneXHelper";  
  
let { width: windowWidth, height: windowHeight } = Dimensions.get("window");  
  
// 全面屏手机 window.height 不包含 statusBar.height 不是真实的屏幕高度  
let { width: screenWidth, height: screenHeight } = Dimensions.get("screen");  
  
// 适配横屏  
if (windowWidth > windowHeight) {  
  [windowWidth, windowHeight] = [windowHeight, windowWidth];  
}  
if (screenWidth > screenHeight) {  
  [screenWidth, screenHeight] = [screenHeight, screenWidth];  
}  
  
//像素比例  
const ratio_750 = windowWidth / 750;  
const ratio_1080 = windowWidth / 1080;  
  
/**  
 * 布局工具  
 */  
export default class Layout {  
  /**  
   * 是否是Android平台  
   */  
  static isAndroid = Platform.OS === "android";  
  /**  
   * 边缘距离  
   */  
  static edgeSize = Layout.DP(32);  
  /**  
   * 分割线高度  
   */  
  static dividerHeight = StyleSheet.hairlineWidth;  
  /**  
   * 标题字体大小  
   */  
  static titleSize = Layout.DP(28);  
  /**  
   * 值字体大小  
   */  
  static valueSize = Layout.DP(26);  
  /**  
   * 子标题字体大小  
   */  
  static subTitleSize = Layout.DP(24);  
  /**  
   * 底部安全距离  
   */  
  static bottomSaveSpace = getBottomSpace();  
  /**  
   * 状态栏高度  
   */  
  static statusBarHeight = getStatusBarHeight(true);  
  /**  
   * 导航栏高度  
   */  
  static navbarHeight = 44;  
  /**  
   * 导航栏+状态栏高度  
   */  
  static totalNavHeight = 44 + getStatusBarHeight(true);  
  /**  
   * 窗口宽度  
   */  
  static windowWidth = windowWidth;  
  /**  
   * 窗口高度  
   */  
  static windowHeight = windowHeight;  
  /**  
   * 屏幕宽度  
   */  
  static screenWidth = screenWidth;  
  /**  
   * 屏幕高度  
   */  
  static screenHeight = screenHeight;  
  /**  
   * px像素转换成react-native用的dp  
   * @param {*} num 像素  
   * @param {*} ratio 像素比例，默认750  
   */  static DP(num, ratio = null) {  
    let _ratio = ratio_750;  
  
    if (ratio) {  
      _ratio = ratio;  
    }  
  
    if (num === 1) {  
      return StyleSheet.hairlineWidth;  
    }  
  
    if (num === 0) {  
      return 0;  
    }  
    return num * _ratio;  
  }  
  /**  
   * px像素转换成react-native用的dp，1080比例  
   * @param {*} num 像素  
   */  
  static DP3(num) {  
    return Layout.DP(num, ratio_1080);  
  }  
}
```