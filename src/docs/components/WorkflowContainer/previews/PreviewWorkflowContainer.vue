<template>
	<div class="pkpWorkflow" v-cloak>
		<pkp-header :is-one-line="true" class="pkpWorkflow__header">
			<h1 class="pkpWorkflow__identification">
				<badge
					v-if="submission.status === getConstant('STATUS_PUBLISHED')"
					class="pkpWorkflow__identificationStatus"
					:is-success="true"
				>
					Published
				</badge>
				<badge
					v-else-if="submission.status === getConstant('STATUS_SCHEDULED')"
					class="pkpWorkflow__identificationStatus"
					:is-primary="true"
				>
					Scheduled
				</badge>
				<span class="pkpWorkflow__identificationId">{{ submission.id }}</span>
				<span class="pkpWorkflow__identificationDivider">/</span>
				<span class="pkpWorkflow__identificationAuthor">
					{{ currentPublication.authorsStringShort }}
				</span>
				<span class="pkpWorkflow__identificationDivider">/</span>
				<span class="pkpWorkflow__identificationTitle">
					{{
						localizeSubmission(
							currentPublication.fullTitle,
							currentPublication.locale
						)
					}}
				</span>
			</h1>
			<template slot="actions">
				<pkp-button
					element="a"
					:label="
						submission.status === getConstant('STATUS_PUBLISHED')
							? i18n.view
							: i18n.preview
					"
					:href="submission.urlPublished"
				></pkp-button>
				<pkp-button
					label="Activity Log"
					ref="activityButton"
					@click="alert('Opens the editorial history in a modal.')"
				></pkp-button>
				<pkp-button
					label="Library"
					ref="library"
					@click="alert('Opens the submission library in a modal.')"
				></pkp-button>
			</template>
		</pkp-header>
		<tabs default-tab="workflow">
			<tab id="workflow" label="Workflow">
				... Workflow ...
			</tab>
			<tab id="publication" label="Publication">
				<div class="pkpPublication" ref="publication" aria-live="polite">
					<pkp-header class="pkpPublication__header">
						<span class="pkpPublication__status">
							<strong>{{ i18n.status }}</strong>
							<span
								v-if="
									workingPublication.status === getConstant('STATUS_QUEUED') &&
										workingPublication.id === currentPublication.id
								"
								class="pkpPublication__statusUnpublished"
							>
								Unscheduled
							</span>
							<span
								v-else-if="
									workingPublication.status === getConstant('STATUS_SCHEDULED')
								"
							>
								Scheduled
							</span>
							<span
								v-else-if="
									workingPublication.status === getConstant('STATUS_PUBLISHED')
								"
								class="pkpPublication__statusPublished"
							>
								Published
							</span>
							<span v-else class="pkpPublication__statusUnpublished">
								Unpublished
							</span>
						</span>
						<span
							v-if="publicationList.length > 1"
							class="pkpPublication__version"
						>
							<strong tabindex="0">{{ i18n.version }}</strong>
							{{ workingPublication.version }}
							<dropdown
								class="pkpPublication__versions"
								label="All Versions"
								:is-link="true"
								submenu-label="Submenu"
							>
								<ul>
									<li
										v-for="publication in publicationList"
										:key="publication.id"
									>
										<button
											class="pkpDropdown__action"
											:disabled="publication.id === workingPublication.id"
											@click="setWorkingPublicationById(publication.id)"
										>
											{{ publication.version }} /
											<template
												v-if="
													publication.status === getConstant('STATUS_QUEUED') &&
														publication.id === currentPublication.id
												"
											>
												Unscheduled
											</template>
											<template
												v-else-if="
													publication.status === getConstant('STATUS_SCHEDULED')
												"
											>
												Scheduled
											</template>
											<template
												v-else-if="
													publication.status === getConstant('STATUS_PUBLISHED')
												"
											>
												{{ publication.datePublished }}
											</template>
											<template v-else>Unpublished</template>
										</button>
									</li>
								</ul>
							</dropdown>
						</span>
						<template slot="actions">
							<pkp-button
								v-if="
									workingPublication.status === getConstant('STATUS_QUEUED')
								"
								ref="publish"
								:label="
									submission.status === getConstant('STATUS_PUBLISHED')
										? i18n.publish
										: i18n.schedulePublication
								"
								@click="alert('Opens a modal to confirm publishing.')"
							></pkp-button>
							<pkp-button
								v-else-if="
									workingPublication.status === getConstant('STATUS_SCHEDULED')
								"
								label="Unschedule"
								:is-warnable="true"
								@click="
									alert('Opens a modoal to confirm scheduling for publication.')
								"
							></pkp-button>
							<pkp-button
								v-else-if="
									workingPublication.status === getConstant('STATUS_PUBLISHED')
								"
								label="Unpublish"
								:is-warnable="true"
								@click="alert('Opens a modal to confirm unpublishing.')"
							></pkp-button>
							<pkp-button
								v-if="canCreateNewVersion"
								ref="createVersion"
								label="Create New Version"
								@click="
									alert(
										'Creates a new version and loads that as the working publication.'
									)
								"
							></pkp-button>
						</template>
					</pkp-header>
					<div
						v-if="workingPublication.status === getConstant('STATUS_PUBLISHED')"
						class="pkpPublication__versionPublished"
					>
						This version has been published and can not be edited.
					</div>
					<tabs
						class="pkpPublication__tabs"
						:is-side-tabs="true"
						:label="publicationTabsLabel"
					>
						<tab id="titleAbstract" label="Title & Abstract">
							<pkp-form v-bind="components.titleAbstract" @set="set" />
						</tab>
						<tab id="contributors" label="Contributors">
							<div id="contributors-grid" ref="contributors">
								<spinner></spinner>
							</div>
						</tab>
						<tab id="metadata" label="Metadata">
							<pkp-form v-bind="components.metadata" @set="set" />
						</tab>
						<tab v-if="supportsReferences" id="citations" label="Citations">
							<pkp-form v-bind="components.citations" @set="set" />
						</tab>
						<tab id="identifiers" label="Identifiers">
							... DOIs ...
						</tab>
						<tab id="galleys" label="Galleys">
							<div id="representations-grid" ref="representations">
								<spinner></spinner>
							</div>
						</tab>
						<tab id="license" label="Permissions & Disclosure">
							... copyright and license ...
						</tab>
						<tab id="issue" label="Issue">
							... issue form for cover image, section, etc...
						</tab>
					</tabs>
					<span class="pkpPublication__mask" :class="publicationMaskClasses">
						<spinner></spinner>
					</span>
				</div>
			</tab>
		</tabs>
	</div>
</template>

<script>
import WorkflowContainer from '@/components/Container/WorkflowContainer.vue';
import formCitations from '@/docs/components/Form/helpers/form-citations';
import formMetadata from '@/docs/components/Form/helpers/form-metadata';
import formTitleAbstract from '@/docs/components/Form/helpers/form-title-abstract';
import basePublication from '@/docs/data/publication';

export default {
	extends: WorkflowContainer,
	data: function() {
		const currentPublication = {
			...basePublication,
			id: 852,
			datePublished: '2019-03-23 09:12:32',
			isPublished: true,
			issueId: 1,
			lastModified: '2019-03-23 09:12:32',
			status: 3,
			version: 2
		};
		const latestPublication = {
			...basePublication,
			id: 921,
			lastModified: '2019-06-03 18:03:32',
			status: 1,
			title: {
				en_US: 'Amet sit dolor ipsum lorem',
				fr_CA: 'FR Amet sit dolor ipsum lorem'
			},
			version: 3
		};
		return {
			components: {
				citations: formCitations,
				metadata: formMetadata,
				titleAbstract: formTitleAbstract
			},
			contributorsGridUrl: 'http://example.org',
			csrfToken: '1234',
			currentPublication: currentPublication,
			publicationFormIds: ['titleAbstract'],
			publicationList: [
				{
					id: 801,
					datePublished: '2019-01-01 12:01:23',
					status: 3,
					version: 1
				},
				{
					id: currentPublication.id,
					datePublished: currentPublication.datePublished,
					status: currentPublication.status,
					version: currentPublication.version
				},
				{
					id: latestPublication.id,
					datePublished: latestPublication.datePublished,
					status: latestPublication.status,
					version: latestPublication.version
				}
			],
			representationsGridUrl: 'http://example.org',
			submission: {
				contextId: 1,
				currentPublicationId: 852,
				dateLastActivity: '2018-11-03 12:34:53',
				dateSubmitted: '2018-08-03',
				id: 732,
				lastModified: '2019-06-03 18:03:32',
				stageId: 5,
				status: 1,
				submissionProgress: 0,
				urlAuthorWorkflow: '/authorDashboard/submission/1',
				urlEditorialWorkflow: '/workflow/access/1',
				urlWorkflow: '/workflow/access/1',
				urlPublished: '/articles/view/1'
			},
			submissionApiUrl: 'http://example.org',
			supportsReferences: true,
			workingPublication: latestPublication,
			i18n: {
				preview: 'Preview',
				publish: 'Publish',
				schedulePublication: 'Schedule for Publication',
				status: 'Status: ',
				view: 'View',
				version: 'Version: '
			}
		};
	},
	methods: {
		alert(msg) {
			alert(msg);
		}
	}
};
</script>
