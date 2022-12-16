# Get started with API documentation plug-in

This topic describes how to get started with the API documentation plug-in.

## <a id="dev-first-app"></a> Add your API entry to the Tanzu Application Platform GUI software catalog

In this section, you will:

- [Learn about API entities of the Software Catalog](#about-app-accs)
- [Add a demo API entity and its related Catalog objects to Tanzu Application Platform GUI](#deploy-your-app)
- [Update your demo API entry](#deploy-your-app)

### <a id="about-app-accs"></a> About API entities

The list of API entities is visible on the left-hand side navigation panel of
Tanzu Application Platform GUI.
It is also visible on the overview page of specific components on the home page.
APIs are a definition of the interface between components.

Their definition is provided in machine-readable ("raw") and human-readable formats.
For more information, see [API plugin documentation](api-docs.hbs.md).

### <a id="deploy-your-app"></a> Add a demo API entity to Tanzu Application Platform GUI software catalog

To add a demo API entity and its related Catalog objects, follow the same steps as registering any
other software catalog entity:

1. Navigate to the home page of Tanzu Application Platform GUI. Click **Home** on the left-side
   navigation bar. Click **REGISTER ENTITY**.

    ![REGISTER button on the right side of the header](../../images/getting-started-tap-gui-5.png)

2. **Register an existing component** prompts you to type a repository URL.
   Type the link to the `catalog-info.yaml` file of your choice or use the following sample
   definition. Save this code block as `catalog-info.yaml`, upload it to the Git repository of your
   choice, and copy the link to `catalog-info.yaml`.

   This demo setup includes a domain called `demo-domain` with a single system called `demo-system`.
   This systems consists of two microservices - `demo-app-ms-1` and `demo-app-ms-1` - and one API
   called `demo-api` that `demo-app-ms-1` provides and `demo-app-ms-2` consumes.

    ```yaml
    apiVersion: backstage.io/v1alpha1
    kind: Domain
    metadata:
      name: demo-domain
      description: Demo Domain for Tanzu Application Platform
      annotations:
        'backstage.io/techdocs-ref': dir:.
    spec:
      owner: demo-team

    ---

    apiVersion: backstage.io/v1alpha1
    kind: Component
    metadata:
      name: demo-app-ms-1
      description: Demo Application's Microservice-1
      tags:
        - microservice
      annotations:
        'backstage.io/kubernetes-label-selector': 'app.kubernetes.io/part-of=demo-app-ms-1'
        'backstage.io/techdocs-ref': dir:.
    spec:
      type: service
      providesApis:
       - demo-api
      lifecycle: alpha
      owner: demo-team
      system: demo-app

    ---

    apiVersion: backstage.io/v1alpha1
    kind: Component
    metadata:
      name: demo-app-ms-2
      description: Demo Application's Microservice-2
      tags:
        - microservice
      annotations:
        'backstage.io/kubernetes-label-selector': 'app.kubernetes.io/part-of=demo-app-ms-2'
        'backstage.io/techdocs-ref': dir:.
    spec:
      type: service
      consumesApis:
       - demo-api
      lifecycle: alpha
      owner: demo-team
      system: demo-app

    ---

    apiVersion: backstage.io/v1alpha1
    kind: System
    metadata:
      name: demo-app
      description: Demo Application for Tanzu Application Platform
      annotations:
        'backstage.io/techdocs-ref': dir:.
    spec:
      owner: demo-team
      domain: demo-domain

    ---

    apiVersion: backstage.io/v1alpha1
    kind: API
    metadata:
      name: demo-api
      description: The demo API for Tanzu Application Platform GUI
      links:
        - url: https://api.agify.io
          title: API Definition
          icon: docs
    spec:
      type: openapi
      lifecycle: experimental
      owner: demo-team
      system: demo-app # Or specify system name of your choice
      definition: |
        openapi: 3.0.1
        info:
          title: Demo API
          description: defaultDescription
          version: '0.1'
        servers:
          - url: https://api.agify.io
        paths:
          /:
            get:
              description: Auto generated using Swagger Inspector
              parameters:
                - name: name
                  in: query
                  schema:
                    type: string
                  example: type_any_name
              responses:
                '200':
                  description: Auto generated using Swagger Inspector
                  content:
                    application/json; charset=utf-8:
                      schema:
                        type: string
                      examples: {}
    ```

3. Paste the link to the `catalog-info.yaml` and click **ANALYZE**. Review the catalog entities and
   click **IMPORT**.

    ![Screenshot of the stage for reviewing the entities to be added to the catalog.](../images/api-plugin-7.png)

4. Navigate to the **API** page by clicking **APIs** on the left-hand side navigation panel.
   The catalog changes and entries are visible for further inspection.
   If you select the system **demo-app**, the diagram appears as follows:

    ![Screenshot of the APIs page. It shows the system diagram for the demo dash app.](../images/api-plugin-8.png)

### <a id="deploy-your-app"></a> Update your demo API entry

To update your demo API entry:

1. To update your demo API entity, click on **demo-api** from the list of available APIs in your
   software catalog and click the **Edit** icon on the **Overview** page.

    ![Screenshot of the overview of demo dash api. The edit button on the card labeled About is framed in red.](../images/api-plugin-9.png)

    It opens the source `catalog-info.yaml` file that you can edit. For example, change the
    `spec.paths.parameters.example` from `type_any_name` to `Tanzu` and save your changes.

2. After you made the edits, Tanzu Application Platform GUI re-renders the API entry with the next
   refresh cycle.

## <a id="validation-api"></a> Validation Analysis of API specifications

This section describes the Validation Analysis card, the data format needed to populate the card, and
how to get automatic scores for your OpenAPI entities.

### <a id="about-validation"></a> About Validation Analysis card

When viewing entities of the kind `API` on the Overview tab, you see the Validation Analysis card
that displays the health of an API through various scoring parameters.

![Screenshot of the Validation Analysis card, which is shown at the bottom-right of the Overview tab.](../../images/getting-started-tap-gui-9.png)

To display the health scores, an API entity must contain the following metadata structure:

```yaml
apiVersion: backstage.io/v1alpha1
kind: API
metadata:
  name: NAME
  description: DESCRIPTION
  apiscores:
    scores:
    - id: documentationReport
      label: "Documentation Health"
      value: 34.375
      valueType: percentage
      status: failed
    - id: securityReport
      label: "Security Score"
      value: 70.0
      valueType: percentage
      status: warning
    - id: openApiHealthReport
      label: "OpenAPI Health"
      value: 89.0625
      valueType: percentage
      status: passed
    scoreDetailsURL:  VALIDATION-REPORT-URL-FOR-MORE-DETAILS
# Other API Entity parameters
```

If an API entity follows this schema, the Validation Analysis card displays helpful information
about the API.

```yaml
    - id:        # Unique ID
      label:     # Descriptive label displayed as a title over the numerical value
      value:     # Any number value
      valueType: # One of the types (percentage or other). Displays the % symbol or none.
      status:    # One of the statuses (passed, warning, or failed). Displays the number in green, yellow, or red.
```

### <a id="automatic-validation"></a> Automatic OpenAPI specification validation

You can receive automatic validation analysis for OpenAPI specifications by using APIx.
Follow the installation using the APIx documentation.
In addition, you must use either [API Auto Registration](../../api-auto-registration/about.hbs.md)
or APIx Design GitOps to automatically generate the API entities in Tanzu Application Platform GUI.

The automatic scoring cannot score or replace API entities created through other methods like regular
GitOps or manual registration.
You might see the following message signaling that the OpenAPI specification was registered with
regular GitOps methods or manual registration:

**Validation analysis is currently unavailable for APIs registered via TAP GUI without being attached to a workload.**
