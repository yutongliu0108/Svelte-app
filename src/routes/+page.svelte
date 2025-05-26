<script>
    import { onMount } from "svelte";
    import { fetchItem } from "@digitalculture/ochre-sdk";
    import { MapLibre, DefaultMarker, Popup } from "svelte-maplibre";
    import * as Table from "$lib/components/ui/table/index.js";
    import { Input } from "$lib/components/ui/input/index.js";
    import { Button } from "$lib/components/ui/button/index.js";
    let data;
    let error;
    let loading = true;
    let itemsArray;
    let markers;
    let originMarks;
    let filterMarks;
    let tableData = [];
    let originTableData = [];
    let filterData = [];
    let filterValue = ""; // 存储输入框的值

    async function fetchData() {
        try {
            const { error, item } = await fetchItem(
                "240e6e06-9d05-4210-aa83-f4190639886d",
                "spatialUnit",
            );

            if (error) {
                throw new Error("Response failed!");
            }
            data = item;
            itemsArray = data.items;
            markers = itemsArray.map((item) => ({
                lngLat: [item.coordinates.longitude, item.coordinates.latitude],
                name: item.identification.label,
            }));
            originMarks = markers;

            for (let i = 0; i < itemsArray.length; i++) {
                const resultObject = itemsArray[i].properties.reduce(
                    (obj, property) => {
                        const label = property.label;
                        const content = property.values?.[0]?.content || "";
                        obj[label] = content; // 将 label 作为键，content 作为值
                        return obj;
                    },
                    {},
                ); // 初始值为空对象
                resultObject["name"] = itemsArray[i].identification.label;
                let resultObject1 = removeSpacesFromKeys(resultObject);
                tableData.push(resultObject1);
            }
            originTableData = tableData;
        } catch (err) {
            error = err;
        } finally {
            loading = false;
        }
    }

    function handleSubmit(event) {
        event.preventDefault(); 
        const form = event.target;
        const inputValue = form.querySelector("input").value; 
        if (inputValue === "") {
            tableData = originTableData;
            markers = originMarks;
        } else {
            console.log("tableData", tableData);
            console.log("originMarks", originMarks);
            tableData = tableData.filter(
                (item) => item.Object_type === inputValue,
            );

            markers = filterByName(
                originMarks,
                tableData.map((item) => item.name),
            );
        }
    }
    function filterByName(arr, names) {
        return arr.filter((item) => names.includes(item.name));
    }
    function removeSpacesFromKeys(obj) {
        return Object.fromEntries(
            Object.entries(obj).map(([key, value]) => [
                key.replace(/ /g, "_"), 
                value,
            ]),
        );
    }

    onMount(fetchData); 
</script>

{#if loading}
    <p>Loading...</p>
{:else if error}
    <p>Error: {error.message}</p>
{:else if data}
    <!-- 渲染数据 -->

    <MapLibre
        zoom={2}
        center={[33.9292, 36.0369]}
        class="h-[400px] rounded-lg shadow-2xl shadow-gray-300"
        style="https://basemaps.cartocdn.com/gl/voyager-gl-style/style.json"
    >
        {#each markers as { lngLat, name }}
            <!-- Unlike the custom marker example, default markers do not have mouse events,
        and popups only support the default openOn="click" behavior -->
            <DefaultMarker {lngLat} draggable={true}>
                <Popup offset={[0, -10]}>
                    <div class="text-lg font-bold">{name}</div>
                </Popup>
            </DefaultMarker>
        {/each}
    </MapLibre>
    <br />
    <form
        class="flex w-full max-w-sm items-center space-x-2"
        on:submit={handleSubmit}
    >
        <Input type="text" placeholder="please input type content" />
        <Button type="submit">submit</Button>
    </form>

    <Table.Root>
        <Table.Header>
            <Table.Row>
                <Table.Head class="w-[100px]">Name</Table.Head>
                <Table.Head>Object type</Table.Head>
                <Table.Head>Museum Number</Table.Head>
                <Table.Head>Full TEO Findspot</Table.Head>
                <Table.Head>Script</Table.Head>
                <Table.Head>Language</Table.Head>
            </Table.Row>
        </Table.Header>
        <Table.Body>
            {#each tableData as tableItem (tableItem)}
                <Table.Row>
                    <Table.Cell class="font-medium">{tableItem.name}</Table.Cell
                    >
                    <Table.Cell>{tableItem.Object_type}</Table.Cell>
                    <Table.Cell>{tableItem.Museum_Number}</Table.Cell>
                    <Table.Cell>{tableItem.Full_TEO_Findspot}</Table.Cell>
                    <Table.Cell>{tableItem.Script}</Table.Cell>
                    <Table.Cell>{tableItem.Language}</Table.Cell>
                </Table.Row>
            {/each}
        </Table.Body>
    </Table.Root>
    <!-- <pre>{JSON.stringify(data, null, 2)}</pre> -->
{/if}
