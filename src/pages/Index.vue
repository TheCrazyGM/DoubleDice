<template>
	<q-page class="flex flex-center">
		<div class="row q-gutter-md gamble-page">
			<q-form @submit="onSubmit" @reset="onReset" class="q-gutter-md">
				<p>Name:</p>
				<q-input filled clearable autofocus v-model="hive_account" label="Your Account *" hint="the account paying for this roll, either Hive or Steem" lazy-rules
				:rules="[ val => val && val.length > 0 || 'PLEASE TYPE A NAME']">
					<template v-slot:prepend>
						<q-icon name="account_box" />
					</template>
				</q-input>
				
				<p>Roll:</p>
				<q-input filled clearable type="number" v-model="dice_roll" label="Roll Number *" hint="the dice roll you're betting on, 1 - 6" lazy-rules
				:rules="[val => val !== null && val !== '' || 'PLEASE TYPE A NUMBER', val => val > 0 || 'NUMBER MUST BE MORE THAN 0', val => val < 7 || 'NUMBER MUST BE LESS THAN 7']">
					<template v-slot:prepend>
						<q-icon name="casino" />
					</template>
				</q-input>

				<p>Bet:</p>
				<q-input filled clearable :suffix="currency" v-model="bet_amount" label="Bet *"
				:rules="[val => val !== null && val !== '' || 'PLEASE TYPE A NUMBER', val => val >= 0.1 || 'MINIMUM BET IS 0.1', val => val <= max_bet || `MAXIMUM BET FOR THIS CURRENCY IS CURRENTLY ${max_bet}`]">
					<template v-slot:prepend>
						<q-icon name="payments" />
					</template>
				</q-input>
				
				<br>
				<q-btn-toggle v-model="currency" toggle-color="primary" :options="[{label: 'HIVE', value: 'HIVE'},
				{label: 'HBD', value: 'HBD'}, {label: 'SIM', value: 'SIM'}, {label: 'ARCHON', value: 'ARCHON'},
				{label: 'HUSTLER', value: 'HUSTLER'}, {label: 'BATTLE', value: 'BATTLE'}]" />
				<br><br>
				<q-btn-toggle v-model="currency" toggle-color="info" :options="[{label: 'STEEM', value: 'STEEM'}, {label: 'UFM', value: 'UFM'}]" />

				<div>
					<q-btn :label="'Gamble '+bet_amount+' '+currency" type="submit" color="positive" />
					<q-btn label="Reset" type="reset" color="negative" flat class="q-ml-sm" />
					<br>
					<span class="text-primary">possible winnings: ~{{(bet_amount*2.7).toFixed(3)}} {{currency}} on 1 dice match || ~{{(bet_amount*7.2).toFixed(3)}} {{currency}} on 2 dice match</span>
				</div>
			</q-form>

			<iframe src="https://titanembeds.com/embed/618206078664179733?defaultchannel=682006094024802435&theme=BetterTitan" height="600" width="600" frameborder="0"></iframe>
		</div>
	</q-page>
</template>

<style scoped lang="stylus">
	p
		font-size 2em
	
	.q-btn-toggle
		margin-top -1em

	span
		margin-top 0.1em
		font-size 0.85em
		font-style italic

	.gamble-page
		justify-content center
</style>

<script>
export default {
	name: 'PageIndex',

	data() {
		return {
			hive_account: '',
			dice_roll: 1,
			bet_amount: 0.100,
			currency: 'HIVE',
			max_bets: {
				hive: 0, hbd: 0, sim: 0, archon: 0, hustler: 0, battle: 0, steem: 0, ufm: 0
			},
			loading: false,
		}
	},

	mounted () {
		let styles = [
			'background: linear-gradient(#110b0c, #c41922)'
			, 'border: 1px solid #333'
			, 'color: white'
			, 'display: block'
			, 'text-shadow: 0 1px 0 rgba(0, 0, 0, 0.3)'
			, 'box-shadow: 3px 3px 3px rgba(0, 0, 0, 0.4)'
			, 'line-height: 40px'
			, 'text-align: center'
			, 'font-weight: bold'
		].join(';');

		if (window.hive_keychain) {
        	hive_keychain.requestHandshake(() => { console.log('%c HiveKeychain Found! ', styles) })
		} else console.error('No \'window.hive_keychain\' Found')

		if (window.steem_keychain) {
        	steem_keychain.requestHandshake(() => { console.log('%c SteemKeychain Found! ', styles) })
		} else console.error('No \'window.steem_keychain\' Found')
		
		this.get_max_bets();

	},

	methods: {
		get_max_bets() {
			this.$q.loading.show({
				message: '<strong>Updating Max Bet Balances</strong><br/><span class="text-primary"><em>hang on...</em></span>'
			})

			console.log('Getting Balances...');

			let self = this;

			hive.api.getAccounts(['doubledice'], function(err, response){
				if (err) console.error(err)
				else {
					self.max_bets.hive = Number(response[0].balance.substr(0, response[0].balance.indexOf(' H'))) / 50;
					self.max_bets.hbd = Number(response[0].sbd_balance.substr(0, response[0].sbd_balance.indexOf(' H'))) / 50;
				}
			});

			let hive_ssc = new SSC('https://api.hive-engine.com/rpc');
			let steem_ssc = new SSC('https://api.steem-engine.com/rpc');

			hive_ssc.find('tokens', 'balances', {account: 'doubledice'}, 1000, 0, [], (err, result) => {
				result.forEach(data => {
					console.log(data.symbol, data.balance)
					switch (data.symbol) {
						case 'SIM':
							self.max_bets.sim = Number(data.balance) / 50;
							break;
						case 'ARCHON':
							self.max_bets.archon = Number(data.balance) / 50;
							break;
						case 'HUSTLER':
							self.max_bets.hustler = Number(data.balance) / 50;
							break;
						case 'BATTLE':
							self.max_bets.battle = Number(data.balance) / 50;
							break;
					}
				})
			}).then(() => {
				steem_ssc.find('tokens', 'balances', {account: 'doubledice'}, 1000, 0, [], (err, result) => {
					result.forEach(data => {
						console.log(data.symbol, data.balance)
						switch (data.symbol) {
							case 'UFM':
								self.max_bets.ufm = Number(data.balance) / 50;
								break;
						}
					})
				}).then(() => {
					self.$q.loading.hide()
				})
			})
		},

		onSubmit() {
			let json_data = ''
			
			if (this.currency == 'STEEM') {
				steem_keychain.requestTransfer(this.hive_account, 'doubledice', Number(this.bet_amount).toFixed(3), this.dice_roll.toString(), 'STEEM', function(response) {
					console.log(response);
				})
			}
			else if (this.currency == 'UFM') {
				steem_keychain.requestSendToken(this.hive_account, 'doubledice', Number(this.bet_amount).toFixed(3), this.dice_roll.toString(), this.currency, function(response) {
					console.log(response);
				})
			}
			else if (this.currency == 'HIVE') {
				hive_keychain.requestTransfer(this.hive_account, 'doubledice', Number(this.bet_amount).toFixed(3), this.dice_roll.toString(), 'HIVE', function(response) {
					console.log(response);
				})
			}
			else {
				hive_keychain.requestSendToken(this.hive_account, 'doubledice', Number(this.bet_amount).toFixed(3), this.dice_roll.toString(), this.currency, function(response) {
					console.log(response);
				})
			}

			this.get_max_bets();
		},

		onReset() {
			this.hive_account = null
			this.dice_roll = 1  
			this.bet_amount = 0
			this.get_max_bets();
		},
	},

	computed: {
			max_bet() {
				let maxbet = 0

				if (this.currency == 'HIVE') {
					this.max_bets.hive > 5 ? maxbet = 5 : maxbet = this.max_bets.hive
				}
				else if (this.currency == 'HBD') {
					maxbet = this.max_bets.hbd
				}
				else if (this.currency == 'SIM') {
					maxbet = this.max_bets.sim
				}
				else if (this.currency == 'ARCHON') {
					this.max_bets.archon > 50 ? maxbet = 50 : maxbet = this.max_bets.archon
				}
				else if (this.currency == 'HUSTLER') {
					maxbet = this.max_bets.hustler
				}
				else if (this.currency == 'BATTLE') {
					maxbet = this.max_bets.battle
				}
				else if (this.currency == 'STEEM') {
					maxbet = 1
				}
				else if (this.currency == 'UFM') {
					maxbet = this.max_bets.ufm
				}

				return maxbet
			}
		},
}
</script>
