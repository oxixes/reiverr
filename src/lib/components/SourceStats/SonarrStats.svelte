<script lang="ts">
	import { getDiskSpace } from '$lib/apis/sonarr/sonarrApi';
	import { SONARR_BASE_URL } from '$lib/constants';
	import { library } from '$lib/stores/library.store';
	import { formatSize } from '$lib/utils.js';
	import SonarrIcon from '../svgs/SonarrIcon.svelte';
	import StatsContainer from './StatsContainer.svelte';
	import StatsPlaceholder from './StatsPlaceholder.svelte';

	export let large = false;

	async function fetchStats() {
		const discSpacePromise = getDiskSpace();
		const { itemsArray } = await $library;
		const availableSeries = itemsArray.filter(
			(item) => item.sonarrSeries && item.sonarrSeries.statistics?.episodeFileCount
		);

		const diskSpaceInfo =
			(await discSpacePromise).find((disk) => disk.path === '/') ||
			(await discSpacePromise)[0] ||
			undefined;

		const spaceOccupied = availableSeries.reduce(
			(acc, series) => acc + (series.sonarrSeries?.statistics?.sizeOnDisk || 0),
			0
		);

		const episodesCount = availableSeries.reduce(
			(acc, series) => acc + (series.sonarrSeries?.statistics?.episodeFileCount || 0),
			0
		);

		return {
			episodesCount,
			spaceLeft: diskSpaceInfo?.freeSpace || 0,
			spaceOccupied,
			spaceTotal: diskSpaceInfo?.totalSpace || 0
		};
	}
</script>

{#await fetchStats()}
	<StatsPlaceholder {large} />
{:then { episodesCount, spaceLeft, spaceOccupied, spaceTotal }}
	<StatsContainer
		{large}
		title="Sonarr"
		subtitle="Shows Provider"
		href={SONARR_BASE_URL}
		stats={[
			{ title: 'Episodes', value: String(episodesCount) },
			{ title: 'Space Taken', value: formatSize(spaceOccupied) },
			{ title: 'Space Left', value: formatSize(spaceLeft) }
		]}
		fillPercentage={((spaceTotal - spaceLeft) / spaceTotal) * 100}
		color="#8aacfd21"
	>
		<SonarrIcon slot="icon" class="absolute opacity-20 p-4 h-full inset-y-0 right-2" />
	</StatsContainer>
{/await}
