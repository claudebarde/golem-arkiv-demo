<script lang="ts">
  import {
    createPublicClient,
    createWalletClient,
    custom,
    http
  } from "@arkiv-network/sdk";
  import { kaolin } from "@arkiv-network/sdk/chains";
  import toast, { type ToastOptions } from "svelte-french-toast";
  import { jsonToPayload, ExpirationTime } from "@arkiv-network/sdk/utils";

  let { userAddress = $bindable<`0x${string}` | null>() } = $props();
  let city = $state("");
  let country = $state("");
  let latitude = $state<number | null>(null);
  let longitude = $state<number | null>(null);
  let submitting = $state(false);

  const toastOptions: ToastOptions = {
    position: "bottom-right",
    style: "background-color: #E0DFDF;"
  };

  const checkData = () => {
    if (
      !city ||
      city.length < 2 ||
      !country ||
      country.length < 2 ||
      latitude === null ||
      latitude === 0 ||
      longitude === null ||
      longitude === 0
    ) {
      return false;
    } else {
      return true;
    }
  };

  const createArkivClients = (account?: `0x${string}`) => {
    if (!window.ethereum) {
      throw new Error("MetaMask not installed");
    }

    const publicClient = createPublicClient({
      chain: kaolin,
      transport: http()
    });

    const walletClient = createWalletClient({
      chain: kaolin,
      transport: custom(window.ethereum),
      account
    });

    return { publicClient, walletClient };
  };

  const updateLocation = async () => {
    submitting = true;
    /**
     * TEST DATA
     * Toulouse, France 43.604547327512556, 1.4429675372763748
     * London, UK 51.5073509, -0.1277583
     * Istanbul, Turkey 41.0082376, 28.9783589
     * Bogota, Colombia 4.7110, -74.0721
     */
    if (!checkData()) {
      toast.error("Missing location data", toastOptions);
      submitting = false;
      return;
    }

    const { walletClient } = createArkivClients(userAddress);
    try {
      toast.loading("Updating location...", {
        ...toastOptions,
        duration: 6000
      });
      const entity = await walletClient.createEntity({
        payload: jsonToPayload({
          city,
          country,
          latitude,
          longitude,
          timestamp: Date.now()
        }),
        contentType: "application/json",
        attributes: [
          { key: "type", value: "location" },
          { key: "timestamp", value: Date.now() }
        ],
        expiresIn: ExpirationTime.fromDays(365)
      });
      console.log({ entity });
      toast.success("Location updated successfully", toastOptions);
    } catch (error) {
      toast.error("Error updating location", toastOptions);
    }
    submitting = false;
    city = "";
    country = "";
    latitude = null;
    longitude = null;
  };
</script>

<style lang="scss">
  .update-location {
    margin-top: 20px;
    display: flex;
    flex-direction: column;
    align-items: center;

    div {
      margin: 5px;
    }
  }
</style>

<div class="update-location">
  <p>Update the location:</p>
  <div>
    <input
      type="text"
      name="city"
      id="new-city"
      placeholder="Enter new city"
      bind:value={city}
    />
    <input
      type="text"
      name="country"
      id="new-country"
      placeholder="Enter new country"
      bind:value={country}
    />
  </div>
  <div>
    <input
      type="number"
      name="latitude"
      id="new-latitude"
      placeholder="Enter new latitude"
      bind:value={latitude}
    />
    <input
      type="number"
      name="longitude"
      id="new-longitude"
      placeholder="Enter new longitude"
      bind:value={longitude}
    />
  </div>
  <button class="button-24" onclick={updateLocation} disabled={submitting}>
    {#if submitting}
      Updating...
    {:else}
      Submit
    {/if}
  </button>
</div>
