<script>
    import { onMount } from 'svelte';

    let eMileage = 48;
    let eBattery = 13.3;
    let eTax = 0.5;
    let fConsumption = 0.7;
    let ePrice = 0;
    let fPrice = 0;

    $: eConsumption = eMileage / eBattery;
    $: ePrice10km = ePrice * eConsumption;
    $: fPrice10km = fPrice * fConsumption;
    $: result = ePrice10km < fPrice10km ? "charge" : "fuel";

    async function fetchEPrice() {
        const now = new Date();
        var res = await fetch(`https://www.elprisetjustnu.se/api/v1/prices/2022/${now.getMonth() + 1}-${now.getDate().toString().padStart(2, "0")}_SE3.json`);
        var ePrices = await res.json();

        let ePrice = null;

        for (let p of ePrices) {
            const pdf = new Date(p.time_start);
            const pdt = new Date(p.time_end);
            const now = new Date();

            if (now > pdf && now < pdt) {
                return p["SEK_per_kWh"];
            }
        }

        return null;
    }

    async function fetchFPrice() {
        var res = await fetch('https://henrikhjelm.se/api/getdata.php?lan=jonkoping-lan');
        var fPrices = await res.json();
        return fPrices['jonkopinglan_OKQ8_JonkopingVarstarondellen_Huskvarna__95'];
    }

    async function run() {
        ePrice = await fetchEPrice() + eTax;
        fPrice = await fetchFPrice();
    }

    onMount(run);
</script>

<h3>Right now it's cheaper to {result}</h3>
<strong>1m fuel: {fPrice10km}</strong><br>
<strong>1m elec: {ePrice10km}</strong><br><br>
Current fuel cost: <input type="text" bind:value="{fPrice}"><br>
Current electricity cost: <input type="text" bind:value="{ePrice}"><br><br>
Car avg fuel consumption l/m: <input type="text" bind:value="{fConsumption}"><br>
Car battery size kwh: <input type="text" bind:value="{eBattery}"><br>
Car avg power mileage km: <input type="text" bind:value="{eMileage}"><br>
Car avg power consumption km/kwh: <input type="text" disabled value="{eConsumption}"><br>
Electricity tax: <input type="text" bind:value="{eTax}">
