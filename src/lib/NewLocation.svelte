<script lang="ts">
  import {
    createPublicClient,
    createWalletClient,
    custom,
    http
  } from "@arkiv-network/sdk";
  import { kaolin } from "@arkiv-network/sdk/chains";
  import { desc, eq } from "@arkiv-network/sdk/query";
  import { jsonToPayload, ExpirationTime } from "@arkiv-network/sdk/utils";

  let { userAddress = $bindable<`0x${string}` | null>() } = $props();
  let city = $state("");
  let country = $state("");
  let latitude = $state<number | null>(null);
  let longitude = $state<number | null>(null);

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
    /**
     * TEST DATA
     * Toulouse, France 43.604547327512556, 1.4429675372763748
     * London, UK 51.5073509, -0.1277583
     * Istanbul, Turkey 41.0082376, 28.9783589
     */
    if (!checkData()) {
      // TODO: display error message to user
      console.log("Missing location data");
      return;
    }

    const { walletClient } = createArkivClients(userAddress);
    const { entityKey } = await walletClient.createEntity({
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
  };
</script>

<style lang="scss">
  .update-location {
    margin-top: 20px;
    display: flex;
    flex-direction: column;
    align-items: center;

    input {
      margin: 5px;
      padding: 8px;
      width: 200px;
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
  <button onclick={updateLocation}>Submit</button>
</div>
