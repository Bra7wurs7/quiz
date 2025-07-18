<script lang="ts">
    interface Category {
        name: string;
        questions: Question[];
    }

    interface Question {
        difficulty: number;
        content: string;
        solution: string;
        solution_img_src: string | undefined;
        img_src: string | undefined;
        audio_src: string | undefined;
    }

    class Participant {
        name: string;
        score = $state(0);

        constructor(name: string) {
            this.name = name;
        }
    }

    let categories: Category[] = generateQuizData(5, 5);

    let participants: Participant[] = [
        new Participant("Gute Gruppe"),
        new Participant("Tolles Team"),
    ];
    let lost_points: Participant = $state(new Participant(""));
    let viewed_question = $state<Question | undefined>(undefined);
    let showsolution = $state<boolean>(false);
    let solved_questions = $state<Question[]>([]);

    function generateQuizData(
        numCategories: number,
        numQuestionsPerCategory: number,
    ): Category[] {
        const categories: Category[] = [];

        for (let i = 0; i < numCategories; i++) {
            const categoryName = String.fromCharCode(97 + i);
            const questions: Question[] = [];

            for (let j = 0; j < numQuestionsPerCategory; j++) {
                questions.push({
                    difficulty: j + 1,
                    content: `This is the placeholder content for question ${j + 1} in category "${categoryName}".`,
                    solution: `This is the placeholder solution for question ${j + 1} in category "${categoryName}".`,
                    img_src:
                        "https://www.watchmojo.com/articles/top-10-star-wars-characters",
                    solution_img_src:
                        "https://upload.wikimedia.org/wikipedia/commons/c/ce/Star_wars2.svg",
                    audio_src: "audio/mother.mp3",
                });
            }

            categories.push({
                name: categoryName,
                questions: questions,
            });
        }

        return categories;
    }

    function on_click_question(question: Question) {
        showsolution = false;
        if (viewed_question?.content === question.content) {
            showsolution = false;
            return (viewed_question = undefined);
        } else {
            return (viewed_question = question);
        }
    }

    function on_click_participant(participant: Participant) {
        let q = [...solved_questions];
        showsolution = true;
        if (viewed_question !== undefined) {
            if (q.includes(viewed_question)) {
                q.splice(
                    q.findIndex(
                        (qu) => qu.content === viewed_question?.content,
                    ),
                    1,
                );
                participant.score -= viewed_question.difficulty;
            } else {
                q.push(viewed_question);
                participant.score += viewed_question.difficulty;
            }
        }
        solved_questions = q;
    }
</script>

<div class="dark_theme app">
    {#each categories as category, i (category.name)}
        <div class="col1 row{i + 1} quiz_category">{category.name}</div>

        {#each category.questions as question, j (question.content)}
            <button
                onclick={() => on_click_question(question)}
                class="col{j + 2} row{i + 1} quiz_question
                    {solved_questions.find(
                    (qa) => qa.content === question.content,
                ) !== undefined
                    ? 'answered'
                    : ''}
                    {question.content === viewed_question?.content
                    ? 'semi_full_screen'
                    : ''}"
            >
                {#if question.content === viewed_question?.content}
                    <div class="question_header">
                        {category.name} - {question.difficulty} Punkte
                    </div>
                    <div class="question_content">
                        {#if !showsolution}
                            {question.content}
                            {#if question.img_src !== undefined}
                                <img src={question.img_src} alt="" />
                            {/if}
                        {/if}
                        {#if showsolution}
                            {question.solution}
                            {#if question.solution_img_src !== undefined}
                                <img src={question.solution_img_src} alt="" />
                            {/if}
                        {/if}
                    </div>
                    {#if question.audio_src !== undefined}
                        <audio controls src={question.audio_src}> </audio>
                    {/if}
                    {#if question.audio_src === undefined && question.img_src === undefined}
                        <div></div>
                    {/if}
                    <div></div>
                {/if}
                {#if question.content !== viewed_question?.content}
                    {question.difficulty} Punkte
                {/if}
            </button>
        {/each}
    {/each}

    <div class="quiz_scoreboard">
        <div>
            {#each participants as participant (participant.name)}
                <button
                    class="quiz_participant pointer"
                    onclick={() => on_click_participant(participant)}
                >
                    <div class="name">{participant.name}</div>
                    <div class="score">
                        {participant.score}
                    </div>
                </button>
            {/each}
            <input class="quiz_participant" />
        </div>
        <button
            class="quiz_participant pointer"
            onclick={() => on_click_participant(lost_points)}
        >
            <div class="name">Verlorene Punkte</div>
            <div class="score">{lost_points.score}</div>
        </button>
    </div>
</div>

<style>
    @font-face {
        font-family: "cstm";
        src: url("/fonts/BagnardSans.otf") format("opentype");
    }

    :root {
        --dark_h: #1d2021;
        --dark: #282828;
        --dark_s: #32302f;

        --dark1: #3c3836;
        --dark2: #504945;
        --dark3: #665c54;
        --dark4: #7c6f64;

        --light_h: #f9f5d7;
        --light: #fbf1c7;
        --light_s: #f2e5bc;

        --light1: #ebdbb2;
        --light2: #d5c4a1;
        --light3: #bdae93;
        --light4: #a89984;

        --red: #fb4934;
        --green: #b8bb26;
        --yellow: #fabd2f;
        --blue: #83a598;
        --purple: #d3869b;
        --aqua: #8ec07c;
        --gray: #928374;
        --orange: #fe8019;

        --red-dim: #cc2412;
        --green-dim: #98971a;
        --yellow-dim: #d79921;
        --blue-dim: #458588;
        --purple-dim: #b16286;
        --aqua-dim: #689d6a;
        --gray-dim: #a89984;
        --orange-dim: #d65d0e;

        --base_size: 3px;
        --large_size: 7px;
        --size_border: var(--border);
        --size_border_wide: var(--double-border-width);
        --size_border_radius: var(--border-radius);
        --size_margin: var(--base_size);
        --size_padding: var(--base_size);
        --size_icon: var(--icon);

        --style_border: var(--border-style);

        --shadow:
            rgba(0, 0, 0, 0.12) 0px 1px 3px, rgba(0, 0, 0, 0.24) 0px 1px 2px;
        --shadow_strong:
            rgba(0, 0, 0, 0.2) 0px 1px 2px, rgba(0, 0, 0, 0.1) 0px 1px 3px;
        --shadow_inset:
            inset rgba(0, 0, 0, 0.3) 0px 1px 2px,
            inset rgba(0, 0, 0, 0.2) 0px 1px 3px;
    }

    .dark_theme {
        --color_border: var(--dark1);
        --color_border_light: var(--dark4);
        --color_background: var(--dark);
        --color_background_soft: var(--dark_s);
        --color_background_hard: var(--dark_h);
        --color_foreground: var(--light);
        --color_foreground_soft: var(--light_s);
        --color_foreground_hard: var(--light_h);

        background-color: var(--dark_h);
        color: var(--color_foreground);
    }

    .app {
        box-sizing: border-box;
        padding: 7px;
        height: 100vh;
        width: 100vw;
        background-color: var(--color_background_hard);
        font-family: cstm;

        display: grid;
        grid-gap: 7px;
        grid-template-rows: repeat(5, 1fr);
        grid-template-columns: 20vw repeat(5, 1fr) 20vw;

        .quiz_category {
            display: flex;
            justify-content: center;
            align-items: center;
            background-color: var(--color_background_soft);
            border-top-right-radius: 7px;
            border-bottom-right-radius: 7px;
            font-size: x-large;
        }

        .quiz_question {
            cursor: pointer;
            display: flex;
            justify-content: center;
            align-items: center;
            background-color: var(--color_background);
            border-radius: 7px;
            border-color: transparent;
            color: var(--color_foreground_soft);
            &.answered {
                background-color: var(--color_background_hard);
            }
            &.semi_full_screen {
                flex-direction: column;
                justify-content: space-between;
            }
            .question_header {
                font-size: large;
            }
            .question_content {
                display: flex;
                flex-direction: column;
                font-size: large;
                flex-grow: 1;
                justify-content: center;
                align-items: center;
            }
            audio {
                width: 100%;
            }
            &:hover {
                border-color: var(--color_border_light);
                &.answered {
                    border-color: var(--color_border);
                }
            }
        }

        .quiz_scoreboard {
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            grid-column: 7;
            grid-row: 1/6;
            background-color: var(--color_background_soft);
            border-top-left-radius: 7px;
            border-bottom-left-radius: 7px;
            padding: 5px;

            .quiz_participant {
                display: flex;
                justify-content: space-between;
                padding: 5px;
                border-radius: 5px;
                font-size: large;
                background-color: var(--color_background);
                margin-bottom: 5px;
                border-width: 2px;
                border-color: transparent;
                width: 100%;

                .name {
                    color: var(--color_foreground);
                }
                .score {
                    color: var(--color_foreground);
                }
                &.pointer {
                    cursor: pointer;
                }
                &:hover {
                    border-color: var(--color_border_light);
                }
            }
        }

        .col1 {
            grid-column: 1;
        }
        .col2 {
            grid-column: 2;
        }
        .col3 {
            grid-column: 3;
        }
        .col4 {
            grid-column: 4;
        }
        .col5 {
            grid-column: 5;
        }
        .col6 {
            grid-column: 6;
        }

        .row1 {
            grid-row: 1;
        }
        .row2 {
            grid-row: 2;
        }
        .row3 {
            grid-row: 3;
        }
        .row4 {
            grid-row: 4;
        }
        .row5 {
            grid-row: 5;
        }

        .semi_full_screen {
            grid-row: 1/6;
            grid-column: 2/7;
            z-index: 5;
        }
    }

    input {
        /* Remove background color and borders */
        background-color: transparent;
        /*border: none;*/
        box-shadow: none;
        box-sizing: border-box;
        width: 100%;

        /* Remove padding and margins */
        margin: 0;

        /* Set font to inherit from parent element */
        font-size: inherit;
        font-family: inherit;

        /* Reset text color, appearance and cursor */
        color: inherit;
        appearance: none;
        -webkit-appearance: none;

        &:focus {
            outline: none;
        }
    }
</style>
