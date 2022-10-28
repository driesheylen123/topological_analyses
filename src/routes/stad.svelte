<script context="module">
    export async function load({fetch}) {
        const res1 = await fetch('/api/OBSOLETE_datapoints.json')
        const res2 = await fetch('/api/OBSOLETE_edges.json')
        const res3 = await fetch('/api/OBSOLETE_feature_datapoints.json')
        if ( res1.ok && res2.ok && res3.ok ) {
            return {
                props: {
                    datapoints: await res1.json(),
                    edges: await res2.json(),
                    feature_datapoints: await res3.json()
                }
            }
        }
    }


</script>

<script>
    export let datapoints
    export let edges
    export let feature_datapoints

    $: console.log(feature_datapoints);
    console.log(datapoints);
    console.log(edges);


    import { selected_datapoints, selected_feature_datapoints } from '../lib/store'
    import { difference, extent } from 'd3';
    import { scaleLinear, scaleOrdinal } from 'd3-scale';
    import { schemeCategory10 } from 'd3-scale-chromatic'
    import NodeLink3DCanvas from '../components/NodeLink3DCanvas.svelte';
    import NodeLinkCanvas from '../components/NodeLinkCanvas.svelte';
    import FeatureList from '../components/FeatureList.svelte'
    import { Input, Col, Container, Row } from 'sveltestrap';


    let feature_list = ["","Acetate","Acetone","X__Saturatie","Albumine","Albumine____","Aldosteron","Alfa_1_globulinen","Alfa_2_globulinen","Alkalische_fosfatasen","Antithrombine","Apolipoproteine_A1","Apolipoproteine_B","Basofielen","Beta_globulinen","Bicarbonaat","Bilirubine_totaal","C_peptide_nuchter","Calcium","Calcium__urine_","Calcium_creatinine_ratio","Chloride_creatinine_ratio","Chloriden","Chloriden__urine_","Cholesterol","CK","Complement_C3","Complement_C3d","Complement_C4","Complement_CH50","Cortisol","Creatinine","Creatinine__mg_dL_","CRP_US","Eosinofielen","Eosinofielen_totaal","Ferritine","Foliumzuur_in_serum","Fosfaat","Fosfaat__urine_","Fosfaat_creat__ratio","Gamma_GT","Gamma_globulinen","Gastrine","Glucose","Haptoglobine","HbA1c__IFCC_","HbA1c_bij_niet_diabeticus","HDL_cholesterol","Hematocriet","IgA","IgE_totaal","IgG","IgG_subklasse_2","IgG_subklasse_3","IgG_subklasse_4","IgM","IJzer","Insuline_nuchter","Kalium","Kalium__urine_","Kalium_creatinine_ratio","Koper","LDH","LDL_cholesterol","Leukocyten","Lipase","Lipoproteine__a_","Lymfocyten","Lymfocyten____","Magnesium","Magnesium__urine_","Magnesium_creat__ratio","Monocyten","Natrium","Natrium__urine_","Natrium_creatinine_ratio","Neutrofiele_segmenten","non_HDL_cholesterol","Osmolaliteit","Osmolaliteit__urine_","pH","Proteine_C_activiteit","Proteine_S_vrij","Renine","Soortelijk_gewicht","Totaal_eiwit","Transaminase_ALT__GPT_","Transaminase_AST__GOT_","Transcortine","Transferrine","Triglyceriden","Trombocyten","Ureum","Urinezuur","Urinezuur__mg_dL_","Urinezuur_creat__ratio","Vrij_plasmacortisol","Zink__serum_"]
    let Profiles = ["", "diagnosed", "non-diagnosed"]
    let individuals = ['','ID.1', ID.2',ID.3',ID.4',ID.4',ID.5',ID.6',ID.7',ID.8',ID.9',ID.10',ID.11',ID.12',ID.13',ID.14',ID.15',ID.16',ID.17',ID.18',ID.19',ID.20','...'ID.1',
    let metabolites_features_list = ['', 'Apolipoproteine A1', 'C reactive protein', 'Calcium', 'Cholesterol', 'Creatinine', 'Gamma glutamyltransferase', 'Glucose', 'Glycated haemoglobine HbA1c', 'HDL-cholesterol', 'IGF 1', 'LDL direct', 'Oestron', 'Phosphate', 'Reumafactor', 'SHBG']

    let lowColor = "#edf8b1"
    let highColor = "#2c7fb8"

    // const classColorScale = (d) => {
    //     return scaleOrdinal(schemeCategory10).range()[d["class"]]
    // }

    let colorScales = {}
    $: {
        metabolites_features_list.forEach((f) => {
            let values = Object.values(datapoints).map((d) => { return d[f] })
            let colorScale = (d) => {
                return scaleLinear().domain(extent(values)).range([lowColor,highColor])(d[f])
            }
            colorScales[f] = colorScale
        })
        colorScales['gender'] = (d) => {
            if (d["gender"] == "M" ) { return "blue" }
            else { return "pink" }
        }
    }
    


    // let colorScale;
    let selectedColorScale = "";
    let selectedIndividual;
    let selectedProfile;


    $: {
        if ( selectedIndividual ) {
            $selected_datapoints = {}
            if ( selectedIndividual != "" ) {
                // selectedColorScale = "eccentricity"
                let individual_objects = Object.values(datapoints)
                                            .filter((d) => { return d.individual == selectedIndividual })
                                            .map(d => { return d["_id"]})
                individual_objects.forEach((ind) => { $selected_datapoints[ind] = true })
            }
        }
    }

    $:console.log(selectedIndividual)
    $:console.log(selectedProfile)

    $: colorScale = colorScales[selectedColorScale]

    $: {
         if ( selectedProfile ) {
            $selected_datapoints = {}
            if ( selectedProfile != "" ) {
                // selectedColorScale = "eccentricity"
                let individual_objects = Object.values(datapoints)
                // d.individual refers to individual column in data, that gets filtered out and the data of _id is returned
                                            .filter((d) => { return d.profile == selectedProfile })
                                            .map(d => { return d["_id"]})
   // The (prof) thing is linked to the select profile button that is defined below!! analogue to individual button
                individual_objects.forEach((prof) => { $selected_datapoints[prof] = true })
            }
        }
    }



    let colorScaleFeature = scaleLinear().domain([-1,0,1]).range(["#296e2b","#ffffbf","#d40808"])


    $: value_difference = (n) => {
        let all_dp = Object.values(datapoints)
                            .sort((a,b) => { return a[n["_key"]] - b[n["_key"]] } )
                            .map(d => { return d[n["_key"]]})

        let selected_dp = Object.values(datapoints)
                                .filter((d) => { return $selected_datapoints[d.id] })
                                .sort((a,b) => { return a[n["_key"]] - b[n["_key"]] } )
                                .map(d => { return d[n["_key"]]})

                                // console.log(selected_dp.length)

        let median_selected = selected_dp[Math.floor(selected_dp.length/2)]

        // console.log(median_selected)
        
        let rest_dp = Object.values(datapoints)
                            .filter((d) => { return ! $selected_datapoints[d.id] })
                            .sort((a,b) => { return a[n["_key"]] - b[n["_key"]] } )
                            .map(d => { return d[n["_key"]]})
        let median_rest = rest_dp[Math.floor(rest_dp.length/2)]
        // console.log(median_rest)


        let q75 = all_dp[Math.floor(all_dp.length*0.75)]
        let q25 = all_dp[Math.floor(all_dp.length*0.25)]
        let iqr = q75-q25

        // console.log(iqr)
        // console.log(all_dp.length)

        // console.log(q75)
        // console.log(q25)





        let difference = (median_selected - median_rest)/iqr
        
        // console.log(difference)

        return difference
    }
// $:console.log(value_difference(n))
// in order to see what the differences are in order to properly map towards an appropriate scale
// $:console.log(difference)

</script>

<h1>STAD</h1>
Colour scheme datapoints:
<Input type="select" name="select" id="colourSchemeSelect" bind:value={selectedColorScale}>
    {#each metabolites_features_list as f}
        <option>{f}</option>
    {/each}
</Input>
Select individual:
<Input type="select" name="select" id="individualSelect" bind:value={selectedIndividual}>
    {#each individuals as ind}
        <option>{ind}</option>
    {/each}
</Input>
Select phenotype group:
<Input type = "select" name="select" id="ProfileColourSelect" bind:value={selectedProfile}>
    {#each Profiles as prof}
    <option>{prof}</option>
    {/each}
</Input>

<Container fluid>
    <Row>
        <Col><NodeLink3DCanvas bind:datapoints={datapoints} edges={edges}
            bind:selected={$selected_datapoints}
            nodecolor_function={(node) => { if ( $selected_datapoints[node.id] ) { return "red" } else { return colorScale(node) }}}/>
        </Col>
        <Col><NodeLinkCanvas bind:datapoints={datapoints} edges={edges}
            bind:selected={$selected_datapoints}
            nodecolor_function={(node) => { if ( $selected_datapoints[node.id] ) { return "red" } else { return colorScale(node) }}}/>
        </Col>
    </Row>
    <Row>
        <Col>
            {#if ( Object.keys($selected_datapoints).length > 0 )}
            <FeatureList bind:datapoints={feature_datapoints}
            order_function={(node) => { return value_difference(node) }}
            color_function={(node) => { return colorScaleFeature(value_difference(node))}} />
            {/if}
        </Col>
    </Row>
</Container>

