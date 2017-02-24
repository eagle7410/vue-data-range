<template>
	<div id="data-range" v-show="range.length">
		<ul class="labels" >
			<li
				v-for="(part, inx) in range"
				v-html="part.label"
				:data-num="inx"
				:class ='classLabel(part, inx)'
			></li>
		</ul>
		<div class="input" id="slider"></div>
	</div>
</template>

<script>

export default {
	props : {
		dataRange : {
			type: Array,
			default: function () {
				return [];
 			}
		},
		widthLabel : {
			type: Number,
			default : 40
		}
	},
	data () {
		return {
			range : [],
			step : 1,
			min : 0,
			val : 0,
			max : 0
		}
	},
	methods : {
		/**
		 * Hook after change value.
		 */
		valChangeAfterHook() {
			let that = this;
			let w  = window;

			if (w.vueEvents) {
				w.vueEvents.$emit('dataRangeChangeValue',  {
					val  : that.val,
					data : that.range[that.val]
				});
			}

		},
		/**
		 * Change value handel.
		 */
		valChange () {
			let that = this;

			that.valChangeAfterHook();
		},
		/**
		 * Get class for label.
		 */
		classLabel (data, num) {
			let c = 'data-range-labels';

			if (this.val == num) {
				c += ' active';
			}

			return c;
		},
	},
	created() {
		let that = this;
		let w = window;

		if (that.dataRange.length) {
			that.range = that.dataRange;
		}

		that.val = 0;
		for (let i = 0; i < that.range.length; ++i) {
			let p = that.range[i];

			if (p.isFirst) {
				that.val = i;
				break;
			}
		}

		that.max = that.range.length - 1;

		if(that.max === -1) {
			that.max = 0;
		}

		that.percent = 100 / that.max;

		if (w.vueEvents) {
			w.vueEvents.$on('cartogramsDataRangeUpdate', function () {

				that.range.map((data, inx) => {
					let $el = $('#year_' + inx);
					let cls = 'empty';

					$el.removeClass(cls);

					if (data && data.isEmpty) {
						$el.addClass(cls);
					}
				});

				that.$forceUpdate();
			});
		}

	},
	mounted () {
		let that = this;
		let labelWidth = that.percent * $('#data-range').width() / 100;
		let $range = $('#data-range > .input').slider({
			value : that.val,
			step  : that.step,
			min   : that.min,
			max   : that.max,
			slide : (ev, ui) =>  {
				
				if (!that.range[ui.value] || that.range[ui.value].isEmpty) {
					return false;
				}

				that.val = ui.value;
			}
		});
		let $labels = $('#data-range > .labels');
		$labels.width($labels.width() + labelWidth);
		$labels.find('li').width(labelWidth);

		let cls = 'years';

		let addYear = (isEmpty, num, isAfter) => {
			let call = (isAfter || false) ? 'after' :  'append';
			let setClass = cls;

			if (isEmpty) {
				setClass += ' empty'
			}

			$range[call]($('<div>', {
				id : 'year_' + num,
				class : setClass,
				style : `width: ${that.percent}%;height: 10px;`
			}));
		}

		for (var i = 0; i< that.max ; ++i) {
			addYear(that.range[i].isEmpty, i);
		}

		addYear(that.range[i].isEmpty, i, true);

	}
}
</script>
