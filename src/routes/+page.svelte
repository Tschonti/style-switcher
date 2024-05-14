<script lang="ts">
    // import env from "../.env.json";
    import { initializeApp } from "firebase/app";
    import { docStore } from "sveltefire";
    import { getFirestore, setDoc } from "firebase/firestore";

    // Your web app's Firebase configuration
    const firebaseConfig = {
        apiKey: process.env.VITE_API_KEY,
        authDomain: "style-switcher.firebaseapp.com",
        projectId: "style-switcher",
        storageBucket: "style-switcher.appspot.com",
        messagingSenderId: "682739298879",
        appId: process.env.VITE_APP_ID,
    };

    // Initialize Firebase
    const app = initializeApp(firebaseConfig);
    const db = getFirestore();
    let text = "";
    let styleOptions = [
        "formal",
        "informal",
        "humorous",
        "motivational",
        "sarcastic",
        "poetic",
        "clickbait",
        "dumbed down",
        "business",
    ];
    let selectedOption = styleOptions[0];
    const text_doc = docStore(db, "generate/1");
    const review_doc = docStore(db, "review/1");

    const handleReview = async () => {
        console.log(text_doc);
        await setDoc(review_doc.ref!, {
            input: $text_doc.text,
            style: $text_doc.style,
            output: $text_doc.output,
        });
    };

    const handleSubmit = async (event: SubmitEvent) => {
        const data = new FormData(event.target as HTMLFormElement);

        await setDoc(text_doc.ref!, {
            text: data.get("text"),
            style: data.get("style"),
        });
    };
</script>

<main>
    <h1 class="bold">style-switcher</h1>
    <form on:submit|preventDefault={handleSubmit}>
        <p class="bold">paste the text of the article here:</p>
        <textarea rows="10" name="text" bind:value={text} on:input></textarea>

        <p class="bold">select the expected style of the text:</p>
        <select name="style" bind:value={selectedOption}>
            {#each styleOptions as option}
                <option value={option}>{option}</option>
            {/each}
        </select>
        <button type="submit">switch style!</button>
    </form>
    <p class="bold">generated output:</p>
    <p class="output">
        {$text_doc?.output ?? "Loading..."}
    </p>

    <button on:click={handleReview}>review text transformation</button>
    <p class="bold">review:</p>
    <p class="output">
        {$review_doc?.review ?? "Loading..."}
    </p>
</main>

<style>
    .output {
        white-space: pre-wrap;
    }
    textarea {
        width: 100%;
    }

    main {
        width: 80%;
        margin-left: auto;
        margin-right: auto;
    }

    .bold {
        font-weight: bold;
    }
</style>
