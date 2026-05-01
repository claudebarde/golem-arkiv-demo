<script lang="ts">
  import { onMount } from "svelte";
  import "leaflet/dist/leaflet.css";
  import { createPublicClient, http } from "@arkiv-network/sdk";
  import { kaolin } from "@arkiv-network/sdk/chains";
  import { eq, desc } from "@arkiv-network/sdk/query";
  import Wallet from "../lib/Wallet.svelte";
  import NewLocation from "$lib/NewLocation.svelte";

  type LocationData = {
    latLong: [number, number];
    city: string;
    country: string;
  };

  const ADMIN_ADDRESS = "0xD15e501aFdF31b81dEA374FF6981338463BA89D1";

  let userAddress = $state<`0x${string}` | null>(null);
  let location: LocationData | null = $state(null); // default location (London)

  let mapElement: HTMLDivElement;

  onMount(async () => {
    // loads data from the blockchain and displays it on the map
    const publicClient = createPublicClient({
      chain: kaolin,
      transport: http()
    });

    const result = await publicClient
      .buildQuery()
      .createdBy(ADMIN_ADDRESS)
      .where(eq("type", "location"))
      .withPayload(true)
      .orderBy(desc("timestamp", "number"))
      .limit(1)
      .fetch();

    location = (() => {
      const DEFAULT_LOCATION: LocationData = {
        latLong: [51.505, -0.09],
        city: "London",
        country: "UK"
      }; // London
      if (result.entities.length > 0) {
        const data = result.entities[0].toJson();
        if (!data.latitude || !data.longitude) {
          return DEFAULT_LOCATION;
        }

        return {
          latLong: [data.latitude, data.longitude],
          city: data.city,
          country: data.country
        };
      } else {
        // returns default location (London)
        return DEFAULT_LOCATION;
      }
    })();

    // loads the map
    const L = await import("leaflet");
    const map = L.map(mapElement).setView(location.latLong, 13);

    L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
      attribution:
        '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
    }).addTo(map);

    // subscribes to entity updates
    const unsubscribe = await publicClient.subscribeEntityEvents({
      onEntityCreated: async event => {
        console.log(`new entity created, owner: ${event.owner}`);
        if (event.owner === ADMIN_ADDRESS) {
          const entity = await publicClient.getEntity(event.entityKey);
          if (entity) {
            const data = entity.toJson();
            if (data.latitude && data.longitude && data.city && data.country) {
              location = {
                latLong: [data.latitude, data.longitude],
                city: data.city,
                country: data.country
              };
              map.setView(location.latLong, 13);
            }
          }
        }
      }
    });

    // Later, stop listening:
    return new Promise(() => () => unsubscribe());
  });
</script>

<style>
  h1 {
    text-align: center;
  }

  #map {
    height: 400px;
    width: 70%;
  }
</style>

<Wallet bind:userAddress />
<h1>Where is Claude?</h1>
{#if location}
  <h3>{location.city}, {location.country}</h3>
{/if}
<div id="map" bind:this={mapElement}></div>
{#if userAddress}
  <NewLocation bind:userAddress />
{/if}
