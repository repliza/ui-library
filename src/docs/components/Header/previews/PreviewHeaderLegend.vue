<template>
	<pkp-header>
		<legend>Data Modelling and Conceptual Modelling</legend>
		<template slot="actions">
			<pkp-button label="Preview" />
			<pkp-button label="Activity Log" />
		</template>
	</pkp-header>
</template>

<script>
import PkpHeader from '@/components/Header/Header.vue';
import PkpButton from '@/components/Button/Button.vue';

export default {
	components: {
		PkpButton,
		PkpHeader
	}
};
</script>
