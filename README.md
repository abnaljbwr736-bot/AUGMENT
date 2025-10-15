// App.js
import React, { useState, useEffect } from 'react';
import { I18nManager, SafeAreaView, View, Text, TouchableOpacity, Image, Modal, Alert, ActivityIndicator, StyleSheet } from 'react-native';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import { WebView } from 'react-native-webview';

// اجبار RTL للعربية عند الحاجة (يمكن إطفاؤه إذا تحب)
const defaultLang = 'ar'; // يمكنك تغييره إلى 'en'
I18nManager.forceRTL(defaultLang === 'ar');

const Stack = createStackNavigator();

// --- إعدادات يمكن تعديلها بسهولة ---
const SITE_URL = 'https://YOUR_WEBSITE_URL'; // <-- استبدل هذا برابط موقعك
const SUBSCRIPTION_PRICE = '$0.90 / سنة';
const SPLASH_LOGO = { uri: 'https://i.imgur.com/yourlogo.png' }; // أو ضع مسار محلي لملف الصورة
// ---------------------------------------

function SplashScreen({ navigation }) {
  useEffect(() => {
    const t = setTimeout(() => {
      navigation.replace('Home');
    }, 1800);
    return () => clearTimeout(t);
  }, [navigation]);

  return (
    <SafeAreaView style={styles.splashContainer}>
      <Image source={SPLASH_LOGO} style={styles.logo} resizeMode="contain" />
      <Text style={styles.splashTitle}>AUGMENT</Text>
    </SafeAreaView>
  );
}

function HomeScreen({ navigation }) {
  const [lang, setLang] = useState(defaultLang);
  const [modalVisible, setModalVisible] = useState(false);
  const [checking, setChecking] = useState(false);
  const [subscribed, setSubscribed] = useState(false); // حالياً تجريبي

  const t = (ar, en) => (lang === 'ar' ? ar : en);

  // مكان للتحقق من حالة الاشتراك (يمكن استدعاء backend لاحقًا)
  const checkSubscription = async () => {
    setChecking(true);
    // هنا استدعاء API حقيقي للتحقق من حالة الاشتراك
    // نموذج تجريبي: لا مشترك حتى يدفع
    setTimeout(() => {
      setChecking(false);
      if (subscribed) {
        navigation.navigate('WebWrapper');
      } else {
        setModalVisible(true);
      }
    }, 800);
  };

  const onPurchase = () => {
    // هنا تضع منطق Google Play Billing أو Stripe
    // حالياً نجري محاكاة نجاح
    Alert.alert(t('شراء', 'Purchase'), t('معالجة الدفع تجريبيًا...','Processing test purchase...'));
    setTimeout(() => {
      setSubscribed(true);
      setModalVisible(false);
      Alert.alert(t('تم','Done'), t('تم تفعيل الاشتراك. الآن يمكنك الدخول.','Subscription activated. You can now access the site.'));
    }, 1200);
  };

  return (
    <SafeAreaView style={styles.container}>
      <Text style={styles.title}>AUGMENT</Text>
      <Text style={styles.subtitle}>{t('شبكتك لعرض الأفلام والمسلسلات','Your movies & series network')}</Text>

      <View style={{flexDirection: 'row', marginTop:18}}>
        <TouchableOpacity onPress={()=>setLang('ar')} style={[styles.langBtn, lang==='ar' && styles.langActive]}>
          <Text style={styles.langText}>العربية</Text>
        </TouchableOpacity>
        <TouchableOpacity onPress={()=>setLang('en')} style={[styles.langBtn, lang==='en' && styles.langActive]}>
          <Text style={styles.langText}>English</Text>
        </TouchableOpacity>
      </View>

      <TouchableOpacity onPress={checkSubscription} style={styles.openBtn}>
        {checking ? <ActivityIndicator color="#fff" /> : <Text style={styles.openBtnText}>{t('افتح التطبيق','Open App')}</Text>}
      </TouchableOpacity>

      <TouchableOpacity onPress={()=>setModalVisible(true)} style={styles.purchaseInfo}>
        <Text style={styles.purchaseText}>{t('الاشتراك السنوي:','Annual subscription:')} {SUBSCRIPTION_PRICE}</Text>
      </TouchableOpacity>

      {/* نافذة الدفع */}
      <Modal visible={modalVisible} transparent animationType="slide">
        <View style={styles.modalWrap}>
          <View style={styles.modalCard}>
            <Text style={styles.modalTitle}>{t('اشترك الآن','Subscribe now')}</Text>
            <Text style={{marginTop:8}}>{t('السعر السنوي:','Annual price:')} {SUBSCRIPTION_PRICE}</Text>
            <TouchableOpacity onPress={onPurchase} style={styles.payBtn}>
              <Text style={{color:'#fff'}}>{t('ادفع الآن','Pay now')}</Text>
            </TouchableOpacity>
            <TouchableOpacity onPress={()=>setModalVisible(false)} style={{marginTop:10}}>
              <Text style={{color:'#555'}}>{t('إلغاء','Cancel')}</Text>
            </TouchableOpacity>
          </View>
        </View>
      </Modal>
    </SafeAreaView>
  );
}

function WebWrapper() {
  return (
    <SafeAreaView style={{flex:1}}>
      <WebView source={{ uri: SITE_URL }} style={{flex:1}} startInLoadingState />
    </SafeAreaView>
  );
}

export default function App(){
  return (
    <NavigationContainer>
      <Stack.Navigator screenOptions={{headerShown:false}}>
        <Stack.Screen name="Splash" component={SplashScreen} />
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="WebWrapper" component={WebWrapper} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

const styles = StyleSheet.create({
  splashContainer: {flex:1, justifyContent:'center', alignItems:'center', backgroundColor:'#02030a'},
  logo: {width:160, height:160, marginBottom:12},
  splashTitle: {color:'#18b0ff', fontSize:28, fontWeight:'700'},
  container: {flex:1, alignItems:'center', padding:20, backgroundColor:'#02030a'},
  title: {color:'#18b0ff', fontSize:28, fontWeight:'700'},
  subtitle: {color:'#9bbcd9', marginTop:6},
  langBtn: {padding:8, marginHorizontal:6, borderRadius:8, borderWidth:1, borderColor:'#2e4a5a'},
  langActive: {backgroundColor:'#0b3b56'},
  langText: {color:'#fff'},
  openBtn: {marginTop:28, backgroundColor:'#18b0ff', paddingHorizontal:24, paddingVertical:12, borderRadius:10},
  openBtnText: {color:'#fff', fontWeight:'600'},
  purchaseInfo: {marginTop:18},
  purchaseText: {color:'#9bbcd9'},
  modalWrap: {flex:1, justifyContent:'center', alignItems:'center', backgroundColor:'rgba(0,0,0,0.5)'},
  modalCard: {width:'85%', backgroundColor:'#fff', padding:18, borderRadius:12, alignItems:'center'},
  modalTitle: {fontSize:18, fontWeight:'700'},
  payBtn: {marginTop:14, backgroundColor:'#18b0ff', paddingVertical:10, paddingHorizontal:24, borderRadius:8}
});
