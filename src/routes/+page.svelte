<script lang="ts">
	interface Player {
		id: string;
		name: string;
		scores: Record<string, number | null>;
	}

	const categories = [
		{ name: '1s', label: '1의 개수' },
		{ name: '2s', label: '2의 개수' },
		{ name: '3s', label: '3의 개수' },
		{ name: '4s', label: '4의 개수' },
		{ name: '5s', label: '5의 개수' },
		{ name: '6s', label: '6의 개수' },
		{ name: 'fourOfAKind', label: '4개 같은 수' },
		{ name: 'fullHouse', label: '풀 하우스' },
		{ name: 'smallStraight', label: '스몰 스트레이트', fixed: 15 },
		{ name: 'largeStraight', label: '라지 스트레이트', fixed: 30 },
		{ name: 'yahtzee', label: '요트', fixed: 50 },
		{ name: 'chance', label: '초이스' }
	];

	let players = $state<Player[]>([]);
	let nextId = $state(0);

	function addPlayer() {
		if (players.length < 10) {
			players = [
				...players,
				{
					id: String(nextId++),
					name: `플레이어 ${players.length + 1}`,
					scores: Object.fromEntries(categories.map((c) => [c.name, null]))
				}
			];
		}
	}

	function removePlayer(id: string) {
		players = players.filter((p) => p.id !== id);
	}

	function updateScore(playerId: string, category: string, value: string) {
		const playerIndex = players.findIndex((p) => p.id === playerId);
		if (playerIndex !== -1) {
			const numValue = value === '' ? null : Math.max(0, parseInt(value) || 0);
			players[playerIndex].scores[category] = numValue;
		}
	}

	function getTopTotal(player: Player): number {
		return (
			(player.scores['1s'] || 0) +
			(player.scores['2s'] || 0) +
			(player.scores['3s'] || 0) +
			(player.scores['4s'] || 0) +
			(player.scores['5s'] || 0) +
			(player.scores['6s'] || 0)
		);
	}

	function getBonus(player: Player): number {
		return getTopTotal(player) >= 63 ? 35 : 0;
	}

	function getTotalScore(player: Player): number {
		let total = getTopTotal(player) + getBonus(player);
		for (const cat of categories) {
			if (
				cat.name !== '1s' &&
				cat.name !== '2s' &&
				cat.name !== '3s' &&
				cat.name !== '4s' &&
				cat.name !== '5s' &&
				cat.name !== '6s'
			) {
				total += player.scores[cat.name] || 0;
			}
		}
		return total;
	}

	function initializeGame() {
		players = [];
		nextId = 0;
		addPlayer();
		addPlayer();
	}

	initializeGame();
</script>

<div class="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 p-4">
	<div class="mx-auto max-w-full">
		<div class="mb-6">
			<h1 class="mb-2 text-center text-3xl font-bold text-indigo-900 md:text-4xl">
				🎲 요트 다이스
			</h1>
			<p class="text-center text-gray-600">점수 기록판</p>
		</div>

		<div class="mb-6 flex flex-col items-center gap-2 sm:flex-row sm:justify-center">
			<button
				onclick={addPlayer}
				disabled={players.length >= 10}
				class="rounded-lg bg-indigo-600 px-6 py-2 text-white transition-colors hover:bg-indigo-700 disabled:bg-gray-400 sm:px-8"
			>
				+ 플레이어 추가 ({players.length}/10)
			</button>
		</div>

		{#if players.length > 0}
			<div class="overflow-x-auto rounded-lg bg-white shadow-lg">
				<div class="min-w-full">
					<div
						class="sticky top-0 z-10 grid gap-1 bg-indigo-600 p-1"
						style="grid-template-columns: 180px repeat({players.length}, 100px);"
					>
						<div
							class="flex items-center justify-center rounded bg-indigo-600 p-2 font-bold text-white"
						>
							카테고리
						</div>
						{#each players as player (player.id)}
							<div class="flex flex-col items-center gap-1 rounded bg-indigo-700 p-2">
								<input
									type="text"
									value={player.name}
									onchange={(e) => {
										const playerIndex = players.findIndex((p) => p.id === player.id);
										if (playerIndex !== -1) {
											players[playerIndex].name = e.currentTarget.value;
										}
									}}
									placeholder="이름"
									class="w-full rounded border border-indigo-300 bg-white px-2 py-1 text-center text-sm font-semibold text-gray-800 focus:outline-none"
								/>
								<span class="text-xs font-bold text-indigo-100">합계: {getTotalScore(player)}</span>
							</div>
						{/each}
						<div
							class="hidden items-center justify-center rounded bg-red-500 p-2 font-bold text-white lg:flex"
						>
							삭제
						</div>
					</div>

					<div class="border-t border-gray-200 bg-blue-50 p-1">
						<div class="font-bold text-blue-900">숫자 섹션</div>
					</div>

					{#each categories.slice(0, 6) as category (category.name)}
						<div
							class="grid gap-1 border-b border-gray-200 p-1"
							style="grid-template-columns: 180px repeat({players.length}, 100px);"
						>
							<div
								class="flex items-center rounded bg-gray-100 px-3 py-2 text-sm font-semibold text-gray-700"
							>
								{category.label}
							</div>
							{#each players as player (player.id)}
								<input
									type="number"
									value={player.scores[category.name] ?? ''}
									onchange={(e) => updateScore(player.id, category.name, e.currentTarget.value)}
									placeholder="0"
									class="rounded border border-gray-300 px-2 py-2 text-center focus:border-indigo-500 focus:outline-none"
									min="0"
								/>
							{/each}
							<div class="hidden lg:block"></div>
						</div>
					{/each}

					<div
						class="grid gap-1 border-b-2 border-gray-300 bg-blue-100 p-1 font-bold"
						style="grid-template-columns: 180px repeat({players.length}, 100px);"
					>
						<div class="flex items-center rounded bg-blue-200 px-3 py-2 text-sm text-blue-900">
							상단 합계
						</div>
						{#each players as player (player.id)}
							<div class="rounded bg-white px-2 py-2 text-center font-bold text-blue-900">
								{getTopTotal(player)}
							</div>
						{/each}
						<div class="hidden lg:block"></div>
					</div>

					<div
						class="grid gap-1 border-b-2 border-gray-300 bg-green-100 p-1 font-bold"
						style="grid-template-columns: 180px repeat({players.length}, 100px);"
					>
						<div class="flex items-center rounded bg-green-200 px-3 py-2 text-sm text-green-900">
							보너스 (63이상)
						</div>
						{#each players as player (player.id)}
							<div class="rounded bg-white px-2 py-2 text-center font-bold text-green-900">
								{getBonus(player)}
							</div>
						{/each}
						<div class="hidden lg:block"></div>
					</div>

					<div class="border-t border-gray-200 bg-purple-50 p-1">
						<div class="font-bold text-purple-900">조합 섹션</div>
					</div>

					{#each categories.slice(6) as category (category.name)}
						<div
							class="grid gap-1 border-b border-gray-200 p-1"
							style="grid-template-columns: 180px repeat({players.length}, 100px);"
						>
							<div
								class="flex items-center rounded bg-gray-100 px-3 py-2 text-sm font-semibold text-gray-700"
							>
								{category.label}
								{#if category.fixed}
									<span class="ml-1 text-xs text-indigo-600">({category.fixed})</span>
								{/if}
							</div>
							{#each players as player (player.id)}
								<input
									type="number"
									value={player.scores[category.name] ?? ''}
									onchange={(e) => updateScore(player.id, category.name, e.currentTarget.value)}
									placeholder="0"
									class="rounded border border-gray-300 px-2 py-2 text-center focus:border-indigo-500 focus:outline-none"
									min="0"
								/>
							{/each}
							<div class="hidden lg:block"></div>
						</div>
					{/each}

					<div
						class="grid gap-1 bg-indigo-600 p-1 font-bold"
						style="grid-template-columns: 180px repeat({players.length}, 100px);"
					>
						<div class="flex items-center rounded bg-indigo-700 px-3 py-2 text-sm text-white">
							총점
						</div>
						{#each players as player (player.id)}
							<div
								class="rounded bg-indigo-700 px-2 py-2 text-center text-lg font-bold text-yellow-300"
							>
								{getTotalScore(player)}
							</div>
						{/each}
						<div class="hidden items-center justify-center lg:flex"></div>
					</div>

					{#if players.length > 0}
						<div class="flex flex-col gap-2 border-t border-gray-200 p-4 lg:hidden">
							{#each players as player (player.id)}
								<button
									onclick={() => removePlayer(player.id)}
									class="rounded-lg bg-red-500 px-4 py-2 text-sm font-semibold text-white transition-colors hover:bg-red-600"
								>
									{player.name} 제거
								</button>
							{/each}
						</div>
					{/if}

					<div
						class="hidden gap-1 border-t border-gray-200 p-1 lg:grid"
						style="grid-template-columns: 180px repeat({players.length}, 100px);"
					>
						<div></div>
						{#each players as player (player.id)}
							<button
								onclick={() => removePlayer(player.id)}
								class="rounded bg-red-500 px-2 py-2 text-sm font-semibold text-white transition-colors hover:bg-red-600"
							>
								삭제
							</button>
						{/each}
					</div>
				</div>
			</div>
		{/if}
	</div>
</div>

<style>
	:global(body) {
		font-family:
			-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
	}
</style>
