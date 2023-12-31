<script setup lang="ts">
import { GlobalComponents } from 'vue';

useHead(useSetHead('Home', 'The Atom web proxy'));

const sw_toast: any = ref(null);
const ready_toast: any = ref(null);

const proxyui = useState('proxyui', () => false);
const browser = useState('browser', () => false);
const obfuscate = useState('obfuscate', () => false);

function submit_url(url: string) {
	let push_url = '/proxy';

	if (proxyui.value) {
		push_url = '/proxy';
	} else if (browser.value) {
		push_url = '/browser';
	} else {
		if (process.client) {
			window.location.href = `/ultraviolet/uv.start.html?url=${url}`;
		}
				
		return;
	}
			
	useRouter().push(addData(push_url, {
		url,
		obfuscate: obfuscate.value
	}));
}

onMounted(() => {
	const toast_sw = sw_toast.value;
	const toast_ready = ready_toast.value;

	const { startSW, load_bundle, is_registered } = useUltraviolet();

	is_registered().then(is => {
		if (is)
			return;

		toast_sw.show();
		toast_sw.stop(20);

		if (process.client) {
			load_bundle(window, () => {
				toast_sw.stop(50);

				startSW().then(() => {
					toast_sw.resume();
			
					setTimeout(() => {
						toast_ready.show();
					}, 1000);
				});
			});
		}
	});
});
</script>

<template>
	<div class="flex justify-center mt-24">
		<div class="flex flex-row items-center card bg-base-200 h-24 p-2" style="width: 40rem;">
			<div class="absolute top-6 left-6">
				<SettingsMenu
					size="md"
					:settings="[
						{
							id: 'cyrillic',
							name: 'Cyrillic Obfuscation',
							type: 'boolean',
							value: obfuscate,
							disabled: !proxyui && !browser,
							onchange: (value: boolean) => {
								obfuscate = value;
							},
						},
						{
							id: 'proxyui',
							name: 'Proxy UI',
							type: 'boolean',
							value: proxyui,
							incompatible: ['browser'],
							onchange: (value: boolean) => {
								proxyui = value;
							},
						},
						{
							id: 'browser',
							name: 'Browser',
							type: 'boolean',
							value: browser,
							incompatible: ['proxyui'],
							onchange: (value: boolean) => {
								browser = value;
							},
						}
					]"
				/>
			</div>

			<div class="inline-flex w-full justify-center">
				<Searchbar ref="searchbar" :submit="submit_url" />
			</div>

			<div>
				<button
					class="btn btn-ghost btn-square absolute right-6 top-6"
					aria-label="Go"
					@click.prevent="(($refs.searchbar as any) as GlobalComponents['Searchbar']).submit_url()"
				>
					<Icon
						name="solar:round-alt-arrow-right-outline"
						color="white"
						size="1.5rem"
					/>
				</button>
			</div>

			<Toast
				id="sw_toast"
				ref="sw_toast"
				icon="solar:info-circle-linear"
				:timeout_ms="1000"
				message="Registering Service Workers..."
				hidden
			/>

			<Toast
				id="ready_toast"
				ref="ready_toast"
				icon="solar:check-circle-linear"
				:timeout_ms="1000"
				message="Ultraviolet Ready!"
				hidden
			/>
		</div>
	</div>
</template>