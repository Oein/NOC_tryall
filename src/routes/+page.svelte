<script lang="ts">
	import { Button, Input, Card } from 'nunui';

	type Vari = {
		name: string;
		min: string;
		max: string;
		step: string;
	};

	let variables: Vari[] = [];
	let formula = '';

	let res: string[] = [];
	const test = () => {
		res = [];
		const rx = variables
			.map((x, i) => {
				const precf = `[${i + 1}] `;
				if (x.name === '') return precf + 'Name is empty';
				if (x.min === '') return precf + 'Min is empty';
				if (x.max === '') return precf + 'Max is empty';
				if (x.step === '') return precf + 'Step is empty';
				// name starts with number
				if (!isNaN(Number(x.name[0]))) return precf + 'Name starts with number';
				// name contains invalid characters
				if (x.name.match(/[^a-zA-Z0-9_]/)) return precf + 'Name contains invalid characters';
				// min is not a number
				if (isNaN(Number(x.min))) return precf + 'Min is not a number';
				// max is not a number
				if (isNaN(Number(x.max))) return precf + 'Max is not a number';
				// step is not a number
				if (isNaN(Number(x.step))) return precf + 'Step is not a number';
				// step is 0
				if (Number(x.step) === 0) return precf + 'Step is 0';
				// min is greater than max
				if (Number(x.min) > Number(x.max)) return precf + 'Min is greater than max';
				return '';
			})
			.filter((x) => x != '');
		if (formula === '') res.push('[Formula] Formula is empty');
		if (rx.length > 0) {
			res = rx;
			return;
		}

		try {
			const functionized: Function = eval(
				`(${variables.map((x) => x.name).join(', ')}) => (${formula}) === true`
			);

			const tryM = (arr: number[]) => {
				if (arr.length == variables.length) {
					try {
						if (functionized(...arr)) res.push(`${arr.join(',\t')}`);
					} catch (e: any) {
						res = [];
						res.push(`[Error] ${e.message || e.toString()}`);
						return false;
					}
					return true;
				}

				for (
					let i = Number(variables[arr.length].min);
					i <= Number(variables[arr.length].max);
					i += Number(variables[arr.length].step)
				) {
					if (!tryM([...arr, i])) {
						return false;
					}
				}

				return true;
			};

			tryM([]);
		} catch (e: any) {
			res = [];
			res.push(`[Error] ${e.message || e.toString()}`);
			return false;
		}
	};
</script>

<div class="row">
	<div class="vari">Variables</div>
	<Button
		primary
		on:click={() => {
			variables = [
				...variables,
				{
					name: '',
					min: '0',
					max: '100',
					step: '1'
				}
			];
		}}>Add variable</Button
	>
</div>

<div class="varis">
	{#each variables as vari}
		<div class="col i">
			<div class="row i">
				<div>
					<Input placeholder="name" bind:value={vari.name} outline fullWidth />
				</div>
				<div>
					<Input placeholder="min" bind:value={vari.min} outline fullWidth />
				</div>
			</div>
			<div class="row i">
				<div>
					<Input placeholder="max" bind:value={vari.max} outline fullWidth />
				</div>
				<div>
					<Input placeholder="step" bind:value={vari.step} outline fullWidth />
				</div>
			</div>
			<div class="btn">
				<Button
					on:click={() => {
						variables = variables.filter((v) => v !== vari);
					}}
					error
					style="width: 100%; box-sizing: border-box; text-align:center">Remove</Button
				>
			</div>
		</div>
	{/each}
</div>

<div class="s" />

<div class="vari">Test formula</div>
<Input placeholder="formula" bind:value={formula} outline fullWidth />

<div class="s" />

<Button on:click={test} secondary style="width: 100%; box-sizing: border-box; text-align:center"
	>Test</Button
>

<div class="s" />

<div class="vari">Result</div>
<Card
	>{#each `Result Count: ${res.length}\n===============\n${res.join('\n')}`.split('\n') as x}<p
			class="li"
		>
			{x}
		</p>{/each}</Card
>

<style>
	.li {
		margin: 0px;
		padding: 0px;
	}
	.s {
		height: 2rem;
	}
	.row,
	.col {
		display: flex;
		flex-direction: row;
		gap: 1rem;
		align-items: center;
	}
	.col {
		flex-direction: column;
	}
	.col.i > .row {
		width: 100%;
	}
	.vari {
		font-size: 1.5rem;
		font-weight: bold;
	}
	.i > * {
		flex-grow: 1;
	}
	.btn {
		flex-grow: 0;
		width: 100%;
	}
	.varis {
		display: flex;
		gap: 1rem;
		flex-direction: column;
	}
</style>
