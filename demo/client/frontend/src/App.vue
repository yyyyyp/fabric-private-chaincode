<!--
Copyright IBM Corp. All Rights Reserved.

SPDX-License-Identifier: Apache-2.0
-->

<template>
  <v-app>
    <v-content>
      <router-view />
    </v-content>
  </v-app>
</template>

<script>
import { mapActions } from "vuex";

export default {
  name: "FPCDemo",

  data() {
    return {
      evtSource: null
    };
  },

  methods: {
    ...mapActions({
      fetchState: "ledger/fetchState",
      newTransactionEvent: "ledger/newTransactionEvent"
    })
  },

  created() {
    document.title = "Fabric Private Chaincode - Demo";

    let that = this;
    // listen for restart then logout
    // note that this currently works with the mock server
    // when using the gateway this listener will output an error but can be ignored; does not harm
    // TODO implement the nofications api at the gateway
    this.evtSource = new EventSource("//localhost:3000/api/notifications");
    this.evtSource.addEventListener("update", event => {
      if (event.data === "restart") {
        if (that.$router.currentRoute.path !== "/debug") {
          location.reload();
        }
      } else {
        that.newTransactionEvent(event);
        that.fetchState();
      }
    });

    this.evtSource.addEventListener("close", () => that.evtSource.close());
  },

  destroyed() {
    this.evtSource.close();
  }
};
</script>
