/* eslint-disable */
<template>
<div class="row justify-content-center">
	<div class="col-md-8">
		<div class="row">
			<div class="col-md-6 mb-3">
				<label for="transacao">Tipo de transação</label>
				<select v-model="transacao" class="form-control custom-select" id="transacao" disabled>
					<option value="venda">Compra / Venda</option>
					<option value="aluguel">Aluguel</option>
				</select>
				<div class="invalid-feedback">
					Selecione um tipo de transação
				</div>
			</div>

			<div class="col-md-6 mb-3">
				<label for="tipo">Tipo de imóvel</label>
				<select v-model="imovel.tipo" class="form-control custom-select" id="tipo" disabled>
					<option value="casa">Casa</option>
					<option value="apartamento">Apartamento</option>
				</select>
				<div class="invalid-feedback">
					Selecione um tipo de imóvel
				</div>
			</div>
		</div>

		<div class="row">
			<div class="col-md-12 mb-3">
				<label for="endereco">Endereço</label>
				<input v-model="imovel.endereco" type="text" class="form-control" id="endereco" @blur="onEnderecoBlur()">
				<div class="invalid-feedback">
					Por favor, digite um endereço válido
				</div>
			</div>
		</div>

		<div class="row">
			<div class="col-md-12 mb-3">
				{{ endereco }}
			</div>
		</div>

		<hr class="mb-4">

		<div class="row">
			<div class="col-md-6 mb-3">
				<label for="tamanho">Tamanho</label>
				<div class="input-group">
					<input v-model="imovel.tamanho" type="text" class="form-control" id="tamanho">
					<div class="input-group-append">
						<span class="input-group-text">m&sup2;</span>
					</div>
				</div>
				<div class="invalid-feedback" style="width: 100%;">
					Digite um tamanho válido
				</div>
			</div>

			<div class="col-md-6 mb-3">
				<label for="quartos">Nº de Quartos</label>
				<input v-model="imovel.quartos" type="number" class="form-control" id="quartos">
				<div class="invalid-feedback">
					Por favor, digite um número válido
				</div>
			</div>
		</div>

		<div class="row">
			<div class="col-md-6 mb-3">
				<label for="banheiros">Nº de Banheiros</label>
				<input v-model="imovel.banheiros" type="number" class="form-control" id="banheiros">
				<div class="invalid-feedback">
					Por favor, digite um número válido
				</div>
			</div>

			<div class="col-md-6 mb-3">
				<label for="suites">Nº de Suítes</label>
				<input v-model="imovel.suites" type="number" class="form-control" id="suites">
				<div class="invalid-feedback">
					Por favor, digite um número válido
				</div>
			</div>
		</div>

		<div class="row">
			<div class="col-md-6 mb-3">
				<label for="vagas">Vagas de Garagem</label>
				<input v-model="imovel.vagas" type="number" class="form-control" id="vagas">
				<div class="invalid-feedback">
					Por favor, digite um número válido
				</div>
			</div>
		</div>

		<hr class="mb-4">

		<div class="row">
			<div class="col-md-6 mb-3">
				<label for="iptu">Valor do IPTU (mensal)</label>
				<div class="input-group">
					<div class="input-group-prepend">
						<span class="input-group-text">R$</span>
					</div>
					<money v-model="imovel.iptu" class="form-control" id="iptu"></money>
				</div>
				<div class="invalid-feedback" style="width: 100%;">
					Digite um valor válido
				</div>
			</div>

			<div class="col-md-6 mb-3">
				<label for="condominio">Valor do Condomínio (mensal)</label>
				<div class="input-group">
					<div class="input-group-prepend">
						<span class="input-group-text">R$</span>
					</div>
					<money v-model="imovel.condominio" class="form-control" id="condominio"></money>
				</div>
				<div class="invalid-feedback" style="width: 100%;">
					Digite um valor válido
				</div>
			</div>
		</div>

		<hr class="mb-4">

		<button class="btn btn-primary btn-lg btn-block" @click="onSubmit()">Obter previsão</button>

		<div v-if="previsao" class="resultado mt-4 pt-4">
			<h4 class="title">
				O valor do imóvel é:
			</h4>
			<h1 class="title previsao">
				{{ previsao.toLocaleString('pt-BR', { style: 'currency', currency: 'BRL'}) }}
			</h1>
		</div>
	</div>
</div>
</template>

<script>
    export default {
		name: 'PrevisaoComponent',
		data() {
			return {
				endereco: '',
				transacao: 'venda',
				imovel: {
					tipo: 'apartamento',
					tamanho: null,
					quartos: null,
					banheiros: null,
					suites: null,
					vagas: null,
					iptu: '',
					condominio: '',
					lat: null,
					lng: null,
				},
				previsao: null,
			};
		},

        mounted() {
        },

		methods: {
			onEnderecoBlur()
			{
				if (!this.imovel.endereco.length) return;

				let instance = this.$axios.create();
				delete instance.defaults.headers.common['X-CSRF-TOKEN'];
				delete instance.defaults.headers.common['X-Requested-With'];

				instance.get('https://maps.googleapis.com/maps/api/geocode/json', { params: {address: this.imovel.endereco + ', Belo Horizonte, Minas Gerais, Brasil', key: 'AIzaSyBjrTvcBLGy5eZb1tdh1FO56qxr4woFI60'} })
					.then((response) => {
						let results = response.data.results;

						if (results && results.length) {
							this.endereco = results[0].formatted_address;
							this.imovel.lat = results[0].geometry.location.lat;
							this.imovel.lng = results[0].geometry.location.lng;
						}
					});
			},

			onSubmit()
			{
				this.$axios.post('/previsao', this.imovel, { params: {transacao: this.transacao} })
					.then((response) => {
						this.previsao = response.data;
					});
			}
		},
    }
</script>

<style scoped>
	.resultado {
		text-align: center;
	}
</style>