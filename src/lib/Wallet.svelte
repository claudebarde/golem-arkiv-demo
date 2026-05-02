<script lang="ts">
  import { kaolin } from "@arkiv-network/sdk/chains";
  import "viem/window";

  let { userAddress = $bindable(), ADMIN_ADDRESS } = $props();
  let hasMetaMask = $state(false);

  const connectWallet = async () => {
    await switchToKaolinChain();
    if (hasMetaMask && window.ethereum) {
      const accounts = await window.ethereum.request({
        method: "eth_requestAccounts"
      });
      if (accounts && accounts.length > 0) {
        userAddress = accounts[0];
      }
    }
  };

  // truncates the address for display purposes
  function truncateAddress(address: string): string {
    return `${address.slice(0, 8)}...${address.slice(-8)}`;
  }

  async function switchToKaolinChain() {
    if (!window.ethereum) {
      throw new Error("MetaMask not installed");
    }

    const chainIdHex = `0x${kaolin.id.toString(16)}`;

    try {
      await window.ethereum.request({
        method: "wallet_switchEthereumChain",
        params: [{ chainId: chainIdHex }]
      });
      hasMetaMask = true;
    } catch (error: unknown) {
      if (
        error &&
        typeof error === "object" &&
        "code" in error &&
        error.code === 4902
      ) {
        // Chain not added yet — add it
        await window.ethereum.request({
          method: "wallet_addEthereumChain",
          params: [
            {
              chainId: chainIdHex,
              chainName: kaolin.name,
              nativeCurrency: kaolin.nativeCurrency,
              rpcUrls: kaolin.rpcUrls.default.http,
              blockExplorerUrls: [kaolin.blockExplorers.default.url]
            }
          ]
        });
        hasMetaMask = true;
      } else {
        throw error;
      }
    }
  }
</script>

<style lang="scss">
  #wallet {
    position: absolute;
    top: 20px;
    right: 20px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    text-align: center;
  }
</style>

<div id="wallet">
  <button class="button-24" onclick={connectWallet}>
    {hasMetaMask && userAddress
      ? truncateAddress(userAddress)
      : "Connect to Kaolin Chain"}
  </button>
  <div>
    {#if userAddress}
      <p>
        Connected to Kaolin Chain {userAddress === ADMIN_ADDRESS
          ? "(Admin)"
          : ""}
      </p>
    {/if}
  </div>
</div>
