<script lang="ts">
	import ChartSimpleIcon from '../lib/icons/ChartSimpleIcon.svelte';
	import HearthPulseIcon from '../lib/icons/HearthPulseIcon.svelte';
	import LinkButton from '../lib/LinkButton.svelte';
	import logo from '$lib/icons/thw-mzgw.webp';
	import FlaskVialIcon from '$lib/icons/FlaskVialIcon.svelte';

	import type { PageData } from './$types';
	import shuffleQuiz from '$lib/shared/stores/shuffleQuiz';
	import { randomInt } from '$lib/utils';
	import BoltIcon from '$lib/icons/BoltIcon.svelte';
	import VestIcon from '$lib/icons/VestIcon.svelte';
	import { onMount } from 'svelte';
	import { bannerMessage } from '$lib/shared/stores/bannerMessage';
	import HammerIcon from '$lib/icons/HammerIcon.svelte';
	import type { QuestionType } from '$lib/model/question';

	const description = {
		headline: 'Inoffizielle Tools für THW-Helfer:',
		subheadline:
			'Grundausbildungs-Quiz, Atemschutz-Quiz, CBRN-Quiz, Finnentest-Tracker & Spannungsfall-Berechnung.',
		keywords: 'THW, Quiz, Finnentest, Spannungsfall, Elektro, CBRN, AGT, Atemschutz, Feuerwehr'
	};

	export let data: PageData;

	function randomQuestionId(questionType: QuestionType) {
		if (!$shuffleQuiz) {
			return 1;
		}

		const questionLength = data.questionTypeLength.get(questionType);
		if (questionLength === undefined) {
			console.log(`QuestionType ${questionType} not found in questionTypeLength`);
			return 1;
		}
		return randomInt(questionLength) + 1;
	}

	onMount(() => {
		const lastVisitBannerVersion = parseInt(
			window.localStorage.getItem('lastVisitBannerVersion') ?? '0'
		);

		let currentActiveMessage =
			'<div>Du hast Ideen für neue Tools, weitere Quizfragen oder Feedback?<br/> Schreib mir gerne über <a href="https://app.thw-messenger.de/thw/app#/contacts/profile/1990855" target="_blank">Hermine</a> (Malte Sehmer).</div>';
		const currentActiveMessageVersion = 2;

		if (currentActiveMessageVersion > lastVisitBannerVersion) {
			if (lastVisitBannerVersion != 0) {
				$bannerMessage = {
					message: currentActiveMessage,
					autoDismiss: {
						duration: 60 * 1000
					}
				};
				window.localStorage.setItem('lastVisitBannerVersion', String(currentActiveMessageVersion));
			} else {
				window.localStorage.setItem('lastVisitBannerVersion', '1');
			}
		}
	});
</script>

<svelte:head>
	<title>THW Tools: Quiz, Finnentest & mehr!</title>
	<meta name="description" content={`${description.headline} ${description.subheadline}`} />
	<meta property="og:title" content="THW Tools: Quiz, Finnentest & mehr!" />
	<meta property="og:description" content={`${description.headline} ${description.subheadline}`} />
	<meta property="og:type" content="website" />
	<meta
		property="og:image"
		content="https://thw-tools.de/_app/immutable/assets/thw-mzgw.24176eee.webp"
	/>
	<meta property="og:locale" content="de_DE" />
	<meta name="keywords" content={description.keywords} />
</svelte:head>

<div class="flex flex-col gap-4 mx-auto h-full items-center justify-between mt-16 max-md:mt-8 py-4">
	<div class="flex flex-col gap-8 items-center px-12 w-full max-w-4xl">
		<img
			src={logo}
			class="w-96 max-md:w-64 aspect-auto"
			width="384"
			height="308.42"
			alt="THW MehrzweckGerätewagen"
		/>
		<h1 class="w-fit text-5xl max-md:text-4xl font-bold">THW Tools</h1>
		<h2 class="w-fit text-2xl max-md:text-lg text-center">
			<span class="font-bold">
				{description.headline}
			</span>
			<span>
				{description.subheadline}
			</span>
		</h2>
		<div class="w-full flex flex-col items-center gap-4 max-w-sm max-md:max-w-[18rem]">
			<LinkButton url={`/quiz/ga/${randomQuestionId('ga')}`} dataUmamiEvent={'Open GA Quiz'}>
				<div class="w-6">
					<HammerIcon />
				</div>
				<div class="font-bold">Grundausbildungs-Quiz</div>
			</LinkButton>
			<LinkButton url={`/quiz/agt/${randomQuestionId('agt')}`} dataUmamiEvent={'Open AGT Quiz'}>
				<div class="w-6">
					<ChartSimpleIcon />
				</div>
				<div class="font-bold">Atemschutz-Quiz</div>
			</LinkButton>
			<LinkButton url={`/quiz/cbrn/${randomQuestionId('cbrn')}`} dataUmamiEvent={'Open CBRN Quiz'}>
				<div class="w-6">
					<FlaskVialIcon />
				</div>
				<div class="font-bold">CBRN-Quiz</div>
			</LinkButton><LinkButton
				url={`/cbrn/protective-suite`}
				dataUmamiEvent={'Open CBRN Protective Suite'}
			>
				<div class="w-6">
					<VestIcon />
				</div>
				<div class="font-bold">CBRN-Schutzanzug</div>
			</LinkButton>
			<LinkButton url="https://finnentest.thw-tools.de" blank dataUmamiEvent={'Open Finnentest'}>
				<div class="w-6">
					<HearthPulseIcon />
				</div>
				<div class="font-bold">Finnentest</div>
			</LinkButton>
			<LinkButton
				url="https://elektro.thw-tools.de"
				blank
				dataUmamiEvent={'Open Elektro Spannungsfall'}
			>
				<div class="w-6">
					<BoltIcon />
				</div>
				<div class="font-bold">Elektro Spannungsfall</div>
			</LinkButton>
			<LinkButton url="/clothing" dataUmamiEvent={'Open THW Clothing'}>
				<div class="w-6">
					<VestIcon />
				</div>
				<div class="font-bold">THW Bekleidungs Rechner</div>
			</LinkButton>
		</div>
	</div>
</div>

{#each ['ga', 'agt', 'cbrn'] as questionType}
	<!-- svelte-ignore a11y-missing-content -->
	<a href={`/quiz/${questionType}`} />
	<!-- svelte-ignore a11y-missing-content -->
	<a href={`/quiz/${questionType}/listing`} />
{/each}
