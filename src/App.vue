<template>
  <router-view />
</template>

<script>
import { defineComponent } from 'vue'
import {setCssVar} from "quasar";
import {
  AdMob,
  AdmobConsentStatus,
  BannerAdPluginEvents,
  BannerAdPosition,
  BannerAdSize, InterstitialAdPluginEvents
} from '@capacitor-community/admob';
import {Capacitor} from "@capacitor/core";
export async function initialize(){
  await AdMob.initialize();


  const [trackingInfo] = await Promise.all([
    AdMob.trackingAuthorizationStatus()
  ]);

  if (trackingInfo.status === 'notDetermined') {
    /**
     * If you want to explain TrackingAuthorization before showing the iOS dialog,
     * you can show the modal here.
     * ex)
     * const modal = await this.modalCtrl.create({
     *   component: RequestTrackingPage,
     * });
     * await modal.present();
     * await modal.onDidDismiss();  // Wait for close modal
     **/

    await AdMob.requestTrackingAuthorization();
  }

  const authorizationStatus = await AdMob.trackingAuthorizationStatus();

  if (
    authorizationStatus.status === 'authorized'
  ) {
    await AdMob.showConsentForm();
  }
}

export async function banner(adId) {
  AdMob.addListener(BannerAdPluginEvents.Loaded, () => {
    // Subscribe Banner Event Listener
  });

  AdMob.addListener(BannerAdPluginEvents.SizeChanged, (size) => {
    // Subscribe Change Banner Size
  });

  const options = {
    adId: adId,
    adSize: BannerAdSize.ADAPTIVE_BANNER,
    position: BannerAdPosition.BOTTOM_CENTER,
    margin: 0,
    // isTesting: true
    // npa: true
  };
  await AdMob.showBanner(options)

}
export async function interstitial(addId) {
  AdMob.addListener(InterstitialAdPluginEvents.Loaded, (info) => {
    // Subscribe prepared interstitial
  });

  const options = {
    adId: addId,
    // isTesting: true
    // npa: true
  };
  await AdMob.prepareInterstitial(options);
  await AdMob.showInterstitial();
}

export default defineComponent({
  name: 'App',
  created() {
    initialize();
    if (Capacitor.getPlatform() === 'android') {
    } else if (Capacitor.getPlatform() === 'ios') {
      banner('ca-app-pub-9515612908682608/9101765672');
    }
  },
  setup() {
    // Setup theme
    setCssVar('primary', '#0a84ff');
    setCssVar('secondary', '#26A69A');
    setCssVar('accent', '#9C27B0');
    setCssVar('dark', '#1d1d1d');
    setCssVar('positive', '#21BA45');
    setCssVar('negative', '#C10015');
    setCssVar('info', '#31CCEC');
    setCssVar('warning', '#F2C037');

    // ...
  }
})
</script>
